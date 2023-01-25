---
title: "Rust - Sobrecarga de operadores (Add)"
date: 2020-05-05T21:05:31+02:00
description: "Como y para que sobrecargar el operador Add"
categories: ["Rust"]
tags: ["Rust"]
image: "abacus.jpg"
---

Te imaginas poder realizar operaciones aritméticas con tus propios tipos en java? poder tener la clase Money y sumarlas entre ellas?
Pues en Rust es posible! vamos a ver un ejemplo sencillo de como implementarlo.

<!--more-->

Para nuestro ejemplo, queremos poder sumar cantidades de una misma moneda, para eso crearemos el ValueObject Money

```rust
#[derive(Debug, PartialEq)]
enum Currency {DOLLAR, EURO}

#[derive(Debug, PartialEq)]
struct Money {
    currency: Currency,
    amount: u8,
}
```
Ahora lo que nos gustaría es poder sumar Money sin tener que sumar el amount interno y volviendo a crear un Money.
En Rust es tan sencillo como implementar el trait std::ops::Add para nuestro struct:

```rust
impl std::ops::Add for Money {
    type Output = Self;

    fn add(self, other: Self) -> Self {
        Money {
            currency: self.currency,
            amount: self.amount + other.amount,
        }
    }
}
```

de esta manera ya podríamos sumar Money entre si:

```rust
#[test]
fn should_add_money_with_same_currency() {
    let ten_dollars = Money {
        currency: Currency::DOLLAR,
        amount: 10,
    };
    let five_dollars = Money {
        currency: Currency::DOLLAR,
        amount: 5,
    };
    let fifteen = Money {
        currency: Currency::DOLLAR,
        amount: 15,
    };
    assert_eq!(ten_dollars + five_dollars, 
               fifteen);
}
```

Pero claro.. qué pasa si estamos sumando dólares con euros? con esta implementación la suma sería incorrecta, con lo que en nuestro caso lo que queremos es que el resultado de la suma sea un Result Type. 
Para ese caso, vamos a cambiar nuestra implementación del Add para que el output pase a ser un Result<Money, E> en lugar de un Money


```rust
impl std::ops::Add for Money {
    type Output = Result<Self, &'static str>;

    fn add(self, money: Self) -> Self::Output {
        if  money.currency != self.currency {
            return Err("Can not operate with different currencies")
        }
        Ok(Money {
            currency: self.currency,
            amount: self.amount + money.amount,
        })
    }
}

#[test]
fn should_add_money_with_same_currency() {
    let ten_dollars = Money {
        currency: Currency::DOLLAR,
        amount: 10,
    };
    let five_dollars = Money {
        currency: Currency::DOLLAR,
        amount: 5,
    };
    let fifteen = ten_dollars + five_dollars;
    assert!(fifteen.is_ok(),true);
    assert_eq!(fifteen.ok().unwrap().amount,15);
}
#[test]
fn should_not_allow_add_money_with_different_currency() {
    let ten_dollars = Money {
        currency: Currency::DOLLAR,
        amount: 10,
    };
    let five_euros = Money {
        currency: Currency::EURO,
        amount: 5,
    };
    let fifteen = ten_dollars + five_euros;
    assert!(fifteen.is_err(),true);
}
```
El Add no es el único operador que tenemos disponible para sobrecargarlo en rust, la lista completa la podemos ver en la [documentación de rust](https://doc.rust-lang.org/std/ops/index.html#traits)

El ejemplo completo de código se puede descargar en [github](https://gist.github.com/jcaromiq/aa4f96856354bb0760dd5b28dbd48ca1#file-add_trait-rs) o ejecutarlo en [Rust Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=abb53e17aa6807b0af4f3b8411ed4bed)
