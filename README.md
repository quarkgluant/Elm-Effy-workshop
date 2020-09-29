# ELM le retour

Pour cete seconde partie nous allons jouer avec [l'éditeur/IDE en ligne](https://ellie-app.com/new) 
et (re)découvrir certains aspects du langage ELM. Une dernière partie sur l'aspect framework ne sera pas abordée ici.

### Rappel
Elm est à la fois un langage **ET** un framework. Ce n'est pas un langage généraliste, il ne sert que pour des app oueb'. C'est un langage typé statiquement et fonctionnel, compilé (transpilé) en javascript dont les points forts sont, outre ceux lié au typage et au fonctionnel, un compilateur très user-friendly et une doc très bien faite.
Il a été créé par un seul homme, Evan Czaplicki et est écrit en Haskell et en Elm. Il a inspiré Vue (notemment Vuex, une solution de gestion des états).
Toutes les données sont immutables.

### Rappel des commandes déjà vues

- `elm init` pour créer une architecture de projet dans un répertoire
- `elm reactor` pour compiler et lancer le serveur de développement dans un répertoire (port 8000 du localhost)
- `elm repl` pour avoir une console Elm (Read Eval Print Loop)

il y a aussi `elm make` pour compiler/transpiler du Elm en JS


### les conditionnels
#### IF
On avait vu le **if** en ligne  
```elm
x = 0
if x < 0 then "too small" else "ok"
```
Pour rappel, un `if`est **toujours** suivi d'un `else`, et les deux expressions doivent avoir le même type

Un autre exemple :
```elm
type Pizza = Calzone | Margherita | QuattroStagione
-- Ici, on définit un type (Pizza) qui peut prendre un des 3 "valeurs"
-- le pipe est comme dans les unions en Typescript, un "ou"


choosePizzaIf : Pizza -> String
-- là, ci-dessus c'est le type de la fonction
choosePizzaIf p =
    if p == Calzone then
        "Pizza chosen: " ++ toString p
    else if p == Margherita then 
        "Pizza chosen: " ++ toString p
    else
        "We don't serve this pizza"
```

il y a sa version multi ligne/cas, ressemblant au case en Ruby, ou au switch en C
```elm
x = 5
if  | x < 0 -> "too small" \ 
    | x > 0 -> "too big" \
    | otherwise -> "just rig
```
pour rappel, l'antislash sert juste à indiquer que l'instruction continue sur les lignes suivantes et n'est plus utile avec la dernière version du REPL,
et, pour info **otherwise** est bien un "mot clé" d'ELM.


#### CASE

l'instruction  `case` est centrale en ELM, comme dans les autres langages fonctionnels car, en plus d'être un switch classique, elle introduit aussi au
**pattern matching**

```elm
type Pizza = Calzone | Margherita | QuattroStagione
-- même exeple que ci-dessus
-- le pipe est comme dans les unions en Typescript, un "ou"


choosePizzaIf : Pizza -> String
-- là, ci-dessus c'est le type de la fonction
choosePizzaIf p =
  case p of
    Calzone
        -> "Pizza chosen: " ++ toString p 
    Margherita
        -> "Pizza chosen: " ++ toString p 
    _
        -> "We don't serve this pizza"
```
le dernier cas, le "else" est figuré par un trait souligné, comme en Ruby pour un argument non utilisé.
Petit exercice: copier/coller la fonction `choosePizzaIf` dans votre REPL. Vous constarez une erreur. A vous d'essayer de la fixer


#### Let-In
La construction Let-In peut s'apparenter aux insctructions d'assignation. Elle permet d'introduire des variables locales qui ne sont utiles et utilisées que pour un bloc, pour alléger la notation

La fonction très simple et très inutile ci-dessous renvoie un **tuple**, c'est-à-dire une structure de données avec 2 ou 3 champs, pouvant chacun être de type différent, comme ici un type de pizza suvi d'un entier. On déclare nos variables en dessous du "let", pour les utiliser dans le bloc "in"
```elm
type Pizza = Calzone | Margherita | QuattroStagione

pizzaOrders : ( Pizza, number ) 
pizzaOrders =
    let
        p = Calzone
        n= 5
    in (p,n)
```
Cette structure Let-In peut être utilisée partout où une expression est valide

Un exemple "un peu" plus compliquée :
```elm
numberOfBombsAround : List Cell -> Int -> Int
numberOfBombsAround cells id =
    let
        leftNeighbours leftCells x =
            List.filter (\cell -> abs ((cell.id - 10) - x) == 1 && cell.isMine) leftCells

        rightNeighbours rightCells x =
            List.filter (\cell -> abs ((cell.id + 10) - x) == 1 && cell.isMine) rightCells

        neighbours asideCells x =
            List.filter (\cell -> abs (cell.id - x) == 1 && cell.isMine) asideCells

        upDownNeighbours upDownCells x =
            List.filter (\cell -> abs (cell.id - x) == 10 && cell.isMine) upDownCells

        inTheGrid inCells =
            List.filter (\cell -> cell.id > 0 && cell.id < 101) inCells
    in
    List.append (leftNeighbours (inTheGrid cells) id) (rightNeighbours (inTheGrid cells) id)
        |> List.append (neighbours (inTheGrid cells) id)
        |> List.append (upDownNeighbours (inTheGrid cells) id)
        |> List.length
```
Détaillons un peu.
ici on déclare une fonction `numberOfBombsAround` qui prend 2 arguments, une liste de cellules et un entier et renvoie un entier
c'est tiré du workshop le démineur, où l'on doit créer un démineur en ELM.
Pour infos, les types sont définis ainsi
```elm
type alias Cell =
    { id : Int, isMine : Bool, revealed : Bool }
-- le type Cell est un alias vers un enregistrement (record) composé de 3 champs

type alias Model =
    { cells : List Cell }
-- le Model est un alias vers un autre enregistrement (record) composé d'une Liste de Cells    
```
Ensuite nous avons un gros bloc Let-In qui permet d'introduire 5 fonctions définies en ligne, toutes utilisant la fonction `filter`, 
équivalente au `select` en Ruby, cete fonction `filter` prenant comme argument une fonction, anonyme ici, et une liste, et retournant une liste.
Deux nouveautés, très utilisées en ELm:
- les fonctions anonymes
 \x -> 2 * x, reconnaissables à l'argument ainsi noté \arg et à leur flèche -> derrière laquelle se trouve la définition de la fonction anonyme
- les Pipes
 ça fait très ésotériques mais c'est en fait très simple, pour éviter d'avoir plein de parenthèses quand vous appliquez plusieurs fonctions. Dans l'exemple ci-dessus, ça remplace
 ```elm
 List.lenght(List.append(List.append( (List.append (List.append (leftNeighbours (inTheGrid cells) id) (rightNeighbours (inTheGrid cells) id)  (neighbours (inTheGrid cells) id)))(upDownNeighbours (inTheGrid cells) id))))
```
Pour vous entraîner à les utiliser
```elm
import Html exposing (text)

main =
    text (divideNumber 5 135)

divideNumber divisor dividend =
    toString (dividend / divisor)
```

copier ce code dans [l éditeur/IDE en ligne](https://ellie-app.com/new) et utiliser des pipes "|>" pour remplacer les parenthèses dans les 2 fonctions, main et divideNumber. On a besoin de la fonction text car une chaîne String n'est pas la même chose qu'une chaine HTML.
Une fois ceci fait utiliser l'autre pipe "<|" qui fait la même chose, mais de droite à gauche


#### les boucles WHILE et FOR
c'est très simple et facile, vous connaissez tous ces instructions en Ruby. En Elm, c'est encore beaucoup plus simple, c'est quasi la même chose, à un "petit" détail près : elles n'existent pas du tout en Elm ! Non pas qu'on n'aie jamais besoin besoin de faire une boucle en Elm, mais dans ce cas on utilise une autre technique, allez, rappelez-vous votre piscine C à 42, où l'on écrivait certaines fonctions de 2 façons, itératives et ... récursives. Hé oui, en Elm pour boucler on utilise que la récursion


## les types
### les types primitifs
Vous connaissez déjà String, Char, Int, Float.

Il y a aussi les types comparables et ceux concaténables

### les structures de données
Il y en a 3 de bases
- les listes
 ["ab", "bcd", "efgh"], [1, 2, 3] tous les élèments doivent être du même type
- les tuples
 (1, 2), (1, 'a'), (1, 'a', "et le reste") ou encore ([1, 2], "ma super chaîne à vélo") sont des tuples valides. Si vous définissez un alias avec un tuple, ou une liste de tuple, tous les autres tuples de ce type devront avoir les mêmes types dans le même ordre. Un tuple peut contenir 9 valeurs au maximum.
- les enregistrements (records)
 { x = 2, y = 7 }  

 Un détail : quand on définit un record sur plusieurs lignes, en Elm on préfère mettre la virgule séparant 2 couples de clé/valeur en début de ligne, comme ça si on veut enlever une valeur, on n'a qu'à modifier une seule ligne
 ```elm
    { x = 2
    , y = 4
    , z = 6
    }
 ```


 Plus les types "classiques" dans la librairie standard d'Elm, les Dict, Array et Set. La principale différence entre un Array et une Liste est qu'avec les arrays on peut travailler sur les indices du tableau (qui commence à zéro).

 ### définition de type
 
 #### Le type Union
 On l'a déjà vu, c'est le | qui sépare des "tags" (commençant par une majuscule), pouvant prendre des paramètres, le tout formant a "tagged union"
 ```elm
type Pizza
    = Calzone Int
    | Margherita Int
    | QuattroStagione Int

>Calzone 5
```
En Elm on parle de constructeur de type. Ici, par exemple Calzone est un constructeur de type, puisqu'il permet de construire de nouvelles instances de Pizza

pour plus d'infos sur les [constructeurs de type en Elm](https://medium.com/elm-shorts/an-intro-to-constructors-in-elm-57af7a72b11e)

(extrait ci-dessous du livre de Bruce Tate, Fred Daoud)
> Seven more languages in seven weeks
#### Building Algebraic Data Types
The beauty and power of a type system becomes much stronger as you build your own complex data types. Take, for example, a chess piece. We need to worry about both color and piece. Use type to define a data type, like this:
```elm
type Color = Black | White
type Piece = Pawn | Knight | Bishop | Rook | Queen | King
type ChessPiece = CP Color Piece
piece = CP Black Queen
>CP Black Queen : ChessPiece
```
Nice. A type constructor allows us to build new instances of a type. Our ChessPiece type consists of the characters CP, our type constructor, followed by a Color and a Piece. Now, we can use case and pattern matching to take the piece apart, like this:
```elm
color = case piece of 
    CP White _ -> White 
    CP Black _ -> Black Black : Color
```
That felt like a little too much work, but we’ll deal with an alternative shortly. Building a type that works like List is a little trickier. You need to know that Cons constructs a list, given an element and another list, like this:
```elm
type List = Nil | Cons Int List
```
This definition is recursive! Cons, which is used at compile time to define types, means construct, with head and tail arguments. We define a type of List as either:
• The type Nil, or
• A list constructed with a head of type Int and a tail of type List
That data type is interesting, but we can do better. We can define an abstract list, one that can hold any data type, like this:
```elm
type List a = Empty | Cons a (List a)
```
In this case, a is some as yet undefined abstract data type. If you’re familiar with Java or JavaScript, think of a as a parametric type parameter, such as T in List<T>, with more flexibility and power. This definition defines a List of a as either:
• Empty, or
• A list constructed with a head of something of type a, and a tail of lists
having items of that same a
If you want to know how a list is evaluated in Elm, look at the data type. You
can represent the type for list [1, 2] as Cons 1 (Cons 2 Empty).
Now when I tell you that you can combine the head of a list with the tail, it makes sense. Cons works on types at compile time. The runtime counterpart of the Cons operator that works on data is ::, and it works just as you’d expect:
\> 1 :: 2 :: 3 :: []
[1,2,3] : [number]
Elm builds the list, right on cue, and then tells us that we’re working with a list of numbers. Brilliant. We’ll dive a little more into types as we move forward. For now, let’s press on. Our chess piece was a little awkward, even if it is reminiscent of Haskell. We can do better. Let’s express a chess piece with another data type, the record.

### les alias
Basiquement un alias de type est un enregistrement (record) avec plusieurs clés/valeurs, et dont le nom comence toujurs par une majuscule. Le modèle défini en Elm est de ce type

```elm
type alias Event =
    { timestamp: Int
    , eventname: String }
type alias Model =
    { xpos : Int
    , ypos : Int
    , numbertones: Int
    , backgroundimage: String , events: List Event
    }
```

### Maybe
En javascript, une fonction peut renvoyer `null`, ce qui peut parfois faire planter l'app. En Elm, ceci est impossible, il n'y a pas de `null`, mais on a `Maybe` qui représente une valeur existante ou pas.

En Elm, toutes les valeurs sont garanties d'être présentes et valides, **sauf** celles entouré d'un Maybe
Ce type est lui-même un type Union avec comme membres Just et Nothing. Nothing correspond au `null`, et Just signifie qu'il y a une valeur valide.


```elm
anyToString : Maybe a -> String 
anyToString arg =
    case arg of
    Just arg -> toString arg 
    Nothing -> "no value"
```
Si vous n'avez pas tout compris, [une bonne adresse](https://thoughtbot.com/blog/maybe-mechanics) sur le Maybe

### les constantes
Il n'y a pas de constantes en Elm, mais on peut facilement imiter le comportement avec une fonction revoyant la dite "constante"

```elm
returnOnly42 =
    42
```

### les annotations de type
Elles sont facultatives en Elm, mais recommandées

```elm
varassign_to_tuple : ( String, number )
addMultiplyNumbers : number -> List number -> number
addPizza : List Pizza -> Pizza -> List Pizza
getPizzaRecordAsString : { a | pz1 : Pizza, pz2 : Pizza } -> String
```
petit exercice, à vous de déchiffrer les types ci-dessus

Autre exercice (souvenir, souvenir) vous vous rappelez de cet exo d'Olivier
```typescript
enum Gender { M, F }
​
const players: Player[] = [
  { name: 'Oli', score: 92, gender: Gender.M, isCool: true },
  { name: 'Matth', score: 30, gender: Gender.M },
  { name: 'Souf', score: 100, gender: Gender.M },
  { name: 'Thilde', score: 3005, gender: Gender.F },
  { name: 'Andrea', score: 288, gender: Gender.F },
];
​
interface Player {
    score: number;
    name: string;
    gender: Gender;
    isCool?: boolean;
}
​
function averageScore(players: Player[] ): number {
  return players.reduce((acc, player) => {
      return acc + player.score
  }, 0) / players.length;
}
​
function bestOf(playerA: Player, playerB: Player): Player | string {
    if (playerA > playerB) {
        return playerA;        
    } else if (playerB > playerA) {
        return playerB;
    } else {
        return 'TIE';
    }
}
​
function playerNames(players: Player[]): string[] {
    return players.map((player) => player.name);
}
​
function hasScoreOver(player: Player, score: number): boolean {
    return player.score > score;
}
​
function logPlayerState(player: Player): void {
    console.log(`${player.name} has ${player.score}`);
}

```

Hé bien on va essayer de l'adapter en Elm !!  
Dans un premier temps, on ne va s'intéresser qu'aux signatures


Et pour terminer, on va passer par la case (obligatoire) Gifs de chats [ici](https://elm-lang.org/examples/cat-gifs) où vous aller décortiquer le code et le modifier pour afficher un bouton avec un look plus sympa (utilisons Bootstrap), changer le texte, bref expérimenter