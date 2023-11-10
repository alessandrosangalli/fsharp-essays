Make the implicit explicit (Essential F#)
```
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

Going Further
```
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

Or
type Customer =
  | Eligible of Id:string
  | Registered of Id:string
  | Guest of Id:string
```
