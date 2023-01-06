# darsec
Daml based parser combinator library.

See:
 - [Parsec: Direct Style Monadic Parser Combinators](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/parsec-paper-letter.pdf)
 - [haskell parsec lib](https://hackage.haskell.org/package/parsec)

Examples can be found in Test.daml, but as an illustration conside parsing a csv file:
```haskell
csvFile : Parser [[Text]]
csvFile = many line

line : Parser [Text]
line =
    do result <- cells
       eol
       pure result

cells : Parser [Text]
cells =
    do first <- implode <$> cellContent
       next <- remainingCells
       pure (first :: next)

remainingCells : Parser [Text]
remainingCells = (char "," >> cells) <|> pure []

cellContent : Parser [Text]
cellContent =
    many (noneOf ",\n")

domain : Parser [Text]
domain = do
  many1 letter
  char "@"
  domain <- many1 letter
  string ".com"
  pure domain

```
We run the parser against a correctly structured file example and extract the domains from user emails; asserting that they all adhere to the same domain name: `"example.com"`.
```haskell
testCsv = script $ do
  let
    (Some (parsedCsv, _)) = parse
      csvFile
      "Login email,Identifier,One-time password,Recovery code,First name,Last name,Department,Location\nrachel@example.com,9012,12se74,rb9012,Rachel,Booker,Sales,Manchester\nlaura@example.com,2070,04ap67,lg2070,Laura,Grey,Depot,London\ncraig@example.com,4081,30no86,cj4081,Craig,Johnson,Depot,London\nmary@example.com,9346,14ju73,mj9346,Mary,Jenkins,Engineering,Manchester\njamie@example.com,5079,09ja61,js5079,Jamie,Smith,Engineering,Manchester\n"
  forA_ (drop 1 parsedCsv)
    $   assert
    .   all (== "example")
    <$> pure
    .   implode
    .   concatMap fst
    .   catOptionals
    .   (parse domain <$>)

```