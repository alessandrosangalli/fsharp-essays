# Make the implicit explicit (Essential F#)
```fsharp
type RegisteredCustomer = {
  Id : string
  IsEligible : bool
}

type UnregisteredCustomer = {
  Id : string
}

type Customer =
  | Registered of RegisteredCustomer
  | Guest of UnregisteredCustomer

let sarah = Guest { Id = "Sarah" } // Customer

let calculateTotal customer spend =
  let discount =
    match customer with
    | Registered c when c.IsEligible && spend >= 100.0M -> spend * 0.1M
    | _ -> 0.0M
  spend - discount
```

# Going Further
```fsharp
type RegisteredCustomer = {
  Id : string
}

type UnregisteredCustomer = {
  Id : string
}

type Customer =
  | Eligible of RegisteredCustomer
  | Registered of RegisteredCustomer
  | Guest of UnregisteredCustomer

let calculateTotal customer spend =
  let discount =
    match customer with
    | Eligible _ when spend >= 100.0M -> spend * 0.1M
    | _ -> 0.0M
  spend - discount
```

## Or
```fsharp
type Customer =
  | Eligible of Id:string
  | Registered of Id:string
  | Guest of Id:string
```

# Composition and Forward Pipe operators
```fsharp
// Customer -> (Customer * decimal)
let getPurchases customer =
  let purchases = if customer.Id % 2 = 0 then 120M else 80M
  (customer, purchases) // Parentheses are optional
  
// (Customer * decimal) -> Customer
let tryPromoteToVip purchases =
  let (customer, amount) = purchases
  if amount > 100M then { customer with IsVip = true }
  else customer
  
// Customer -> Customer
let increaseCreditIfVip customer =
  let increase = if customer.IsVip then 100M else 50M
  { customer with Credit = customer.Credit + increase }

// Composition operator
let upgradeCustomerComposed = // Customer -> Customer
  getPurchases >> tryPromoteToVip >> increaseCreditIfVip

// Forward pipe operator
let upgradeCustomerPiped customer = // Customer -> Customer
  customer
  |> getPurchases
  |> tryPromoteToVip
  |> increaseCreditIfVip
```

# Scope rules
## Use the same Random instance every time
```fsharp
// unit -> int
let rnd () =
  let rand = Random()
  rand.Next(100)

List.init 50 (fun _ -> rnd())
```

## Use a different Random instance every time
```fsharp
// unit -> int
let rnd =
  let rand = Random()
  fun () -> rand.Next(100)
List.init 50 (fun _ -> rnd())
```

# Null when dotnet interop
## To convert from .Net to an F# Option type, we can use the Option.ofObj and Option.ofNullable functions:
```fsharp
let fromNullObj = Option.ofObj nullObj
let fromNullPri = Option.ofNullable nullPri
```

## To convert from an Option type to .Net types, we can use the Option.toObj and Option.toNullable functions.
```fsharp
let toNullObj = Option.toObj fromNullObj
let toNullPri = Option.toNullable fromNullPri
```

# Exception handling to Result
```fsharp
// decimal -> decimal -> Result<decimal,exn>
let tryDivide (x:decimal) (y:decimal) =
  try
    Ok (x/y)
  with
  | :? DivideByZeroException as ex -> Error ex

// Error "System.DivideByZeroException: Attempted to divide by zero..."
let badDivide = tryDivide 1M 0M
let goodDivide = tryDivide 1M 1M // Some 1M
```

# Function Composition With Result
```fsharp
let upgradeCustomer customer =
  customer
  |> getPurchases
  |> function
    | Ok x -> Ok (tryPromoteToVip x)
    | Error ex -> Error ex
  |> function
    | Ok x -> increaseCreditIfVip x
    | Error ex -> Error ex

let map f result =
  match result with
  | Ok x -> Ok (f x)
  | Error ex -> Error ex

let bind f result =
  match result with
  | Ok x -> f x
  | Error ex -> Error ex

let upgradeCustomer customer =
  customer
  |> getPurchases
  |> Result.map tryPromoteToVip
  |> Result.bind increaseCreditIfVip
```
# Forward pipe tuple
```fsharp
let getTotal items =
  (0M, items) ||> List.fold (fun acc (q, p) -> acc + decimal q * p)
```

# Lists
```fsharp
let list1 = [1..5]
let list2 = [3..7]
let joined = list1 @ list2 // [1;2;3;4;5;3;4;5;6;7]
let joinedEmpty = list1 @ emptyList // [1;2;3;4;5]
let emptyJoined = emptyList @ list1 // [1;2;3;4;5]

let items = [2;5;3;1;4]
let items = [ for x in 1..5 do x ]
let extendedItems = 6::items // [6;1;2;3;4;5]

let readList items =
  match items with
  | [] -> "Empty list"
  | [head] -> $"Head: {head}" // list containing one item
  | head::tail -> sprintf "Head: %A and Tail: %A" head tail

let emptyList = readList [] // "Empty list"
let multipleList = readList [1;2;3;4;5] // "Head: 1 and Tail: [2;3;4;5]"
let singleItemList = readList [1] // "Head: 1"
```

# List exercise
```fsharp
let addItem item order =
  // 1 - Prepend new item to existing order items
  // 2 - Consolidate each product
  // 3 - Sort items in order by productid to make equality simpler
  // 4 - Update order with new list of items

let recalculate items =
  items
  |> List.groupBy (fun i -> i.ProductId)
  |> List.map (fun (id, items) ->
    { ProductId = id; Quantity = items |> List.sumBy (fun i -> i.Quantity) })
  |> List.sortBy (fun i -> i.ProductId)

let addItems items order =
  let items =
    items @ order.Items
    |> recalculate
  { order with Items = items }

let removeProduct productId order =
  let items =
    order.Items
    |> List.filter (fun x -> x.ProductId <> productId)
    |> List.sortBy (fun i -> i.ProductId)
  { order with Items = items }

let reduceItem productId quantity order =
  let items =
    { ProductId = productId; Quantity = -quantity } :: order.Items
    |> recalculate
    |> List.filter (fun x -> x.Quantity > 0)
  { order with Items = items }

let clearItems order =
  { order with Items = [] }
```

# Active Patterns
```fsharp
let (|ValidDate|_|) (input:string) =
  match DateTime.TryParse(input) with
  | true, value -> Some value
  | false, _ -> None

let parse input =
  match input with
  | ValidDate dt -> printfn "%A" dt
  | _ -> printfn $"'%s{input}' is not a valid date"
```

# Parcial Active Patterns
```fsharp
let (|IsDivisibleBy|_|) divisor n =
  if n % divisor = 0 then Some () else None

let calculate i =
  match i with
  | IsDivisibleBy 3 & IsDivisibleBy 5 -> "FizzBuzz"
  | IsDivisibleBy 3 -> "Fizz"
  | IsDivisibleBy 5 -> "Buzz"
  | _ -> i |> string
```

# Multi-Case Active Patterns
```fsharp
type Rank = Ace|Two|Three|Four|Five|Six|Seven|Eight|Nine|Ten|Jack|Queen|King
type Suit = Hearts|Clubs|Diamonds|Spades
type Card = Rank * Suit

let (|Red|Black|) (card:Card) =
  match card with
  | (_, Diamonds) | (_, Hearts) -> Red
  | (_, Clubs) | (_, Spades) -> Black

let describeColour card =
  match card with
  | Red -> "red"
  | Black -> "black"
  |> printfn "The card is %s"
```

# Single-Case Active Patterns
```fsharp
// string -> int
let (|CharacterCount|) (input:string) =
  input.Length

// string -> bool
let (|ContainsANumber|) (input:string) =
  input
  |> Seq.filter Char.IsDigit
  |> Seq.length > 0

let (|IsValidPassword|) input =
  match input with
  | CharacterCount len when len < 8 -> (false, "Password must be at least 8 characters.")
  | ContainsANumber false -> (false, "Password must contain at least 1 digit.")
  | _ -> (true, "")

let setPassword input =
  match input with
  | IsValidPassword (true, _) as pwd -> Ok pwd
  | IsValidPassword (false, failureReason) -> Error $"Password not set: %s{failureReason}"

let badPassword = setPassword "password"
let goodPassword = setPassword "passw0rd"
```
