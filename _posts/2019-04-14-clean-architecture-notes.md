---
layout: post
title: Clean Architecture - Notes
date: 2019-04-14 22:40:12 +0300
access: private
comments: true
categories: [architecture]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://habr.com/ru/company/mobileup/blog/335382/>

### Заблуждение: Слои, а не сущности

> Request/ResponseModel – просто DTO для передачи данных между слоями.
> По правилу зависимости они должны лежать во внутреннем слое, что мы
> и видим на картинке.

### Доступ к Repository/Gateway только через Interactor?

> В идеале использовать Repository нужно только через Interactor.
>
> В идеале надо делать запросы через Interactor, но также считаю, что
> в небольших проектах, где вероятность добавления логики в Interactor
> ничтожно мала, можно этим правилом поступиться.

it's okay to use `Loader` and `Mutator` directly in contexts without
intermediate operations.

### Заблуждение: Обязательность маппинга между слоями

> А можно использовать DTO из слоя Entities везде во внешних слоях.
> Конечно, если те могут его использовать. Нарушения Dependency Rule
> тут нет.

> Но если вы работаете над проектом один, и это простое приложение,
> то не усложняйте (жизнь) лишним маппингом.

### Заблуждение: маппинг в Interactor’e

> So when we pass data across a boundary, it is always in the form
> that is most convenient for the inner circle.

> Поэтому в Interactor данные должны попадать уже в нужном ему виде.
