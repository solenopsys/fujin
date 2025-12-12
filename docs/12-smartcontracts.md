# 12. Базовый язык (.fjc)

## Что такое .fjc файлы

**`.fjc`** (Fujin Core) — базовый уровень языка Fujin. Это минимальный детерминированный язык без float типов и аннотаций, предназначенный для смартконтрактов и критичной логики.

`.fjc` — это фундамент, на котором строятся все остальные типы файлов (`.fjt`, `.fjs`, `.fjx`).

---

## Зачем нужен отдельный тип файлов

### Проблема с float в смартконтрактах

Float-арифметика недетерминирована:
- Разные процессоры могут давать разные результаты
- Округления зависят от платформы
- Невозможно достичь консенсуса в распределённых системах
- Финансовые операции требуют абсолютной точности

### Решение — .fjc без float

```fujin
// ❌ Нельзя в .fjc
let price: f64 = 19.99
let tax: f64 = price * 0.2

// ✓ Правильно в .fjc
let priceInCents: u64 = 1999  // $19.99 в центах
let taxInCents: u64 = (priceInCents * 20) / 100  // 20%
```

---

## Возможности .fjc

### Что доступно

**Базовые возможности:**
- Структуры данных (объекты, массивы)
- Акторы
- `emit` для сообщений
- Условия (`if-else`, `switch`)
- Единственный цикл `for`
- Целочисленная арифметика (u8-u64, i8-i64, bool, byte, string)
- Логические операторы (&&, ||, !)
- Операторы сравнения (===, !==, <, >, <=, >=)
- Базовые операции с массивами и строками

### Что НЕДОСТУПНО

- **Float типы (f32, f64)** — главное ограничение для детерминированности
- Расширенные математические функции
- Расширенные методы массивов/строк
- JSX-синтаксис
- Каскадные обработчики

---

## Примеры смартконтрактов

### Простой токен

```fujin
// token.fjc
type @transfer(Transfer) = {
  from: u64
  to: u64
  amount: u64
}

type @balance(Balance) = {
  owner: u64
  amount: u64
}

type @transferResult(TransferResult) = {
  success: bool
  fromBalance: u64
  toBalance: u64
}

actor @transfer(msg) {
  const fromBalance = getBalance(msg.from)
  const toBalance = getBalance(msg.to)
  
  if (fromBalance >= msg.amount) {
    const newFromBalance = fromBalance - msg.amount
    const newToBalance = toBalance + msg.amount
    
    setBalance(msg.from, newFromBalance)
    setBalance(msg.to, newToBalance)
    
    emit @transferResult({
      success: true,
      fromBalance: newFromBalance,
      toBalance: newToBalance
    })
  } else {
    emit @transferResult({
      success: false,
      fromBalance: fromBalance,
      toBalance: toBalance
    })
  }
}
```

### Голосование (DAO)

```fujin
// voting.fjc
type @vote(Vote) = {
  proposalId: u64
  voter: u64
  support: bool
}

type @proposal(Proposal) = {
  id: u64
  votesFor: u64
  votesAgainst: u64
  deadline: u64
  executed: bool
}

type @voteResult(VoteResult) = {
  proposalId: u64
  votesFor: u64
  votesAgainst: u64
  canExecute: bool
}

type @error(Error) = {
  message: string
}

actor @vote(msg) {
  const proposal = getProposal(msg.proposalId)
  const currentTime = getBlockTime()
  
  if (currentTime > proposal.deadline) {
    emit @error({ message: "Voting period ended" })
    return
  }
  
  if (hasVoted(msg.proposalId, msg.voter)) {
    emit @error({ message: "Already voted" })
    return
  }
  
  let newVotesFor = proposal.votesFor
  let newVotesAgainst = proposal.votesAgainst
  
  if (msg.support) {
    newVotesFor = newVotesFor + 1
  } else {
    newVotesAgainst = newVotesAgainst + 1
  }
  
  updateProposal(msg.proposalId, newVotesFor, newVotesAgainst)
  markAsVoted(msg.proposalId, msg.voter)
  
  const totalVotes = newVotesFor + newVotesAgainst
  const quorum: u64 = 100
  const canExecute = totalVotes >= quorum && newVotesFor > newVotesAgainst
  
  emit @voteResult({
    proposalId: msg.proposalId,
    votesFor: newVotesFor,
    votesAgainst: newVotesAgainst,
    canExecute: canExecute
  })
}
```

### Мультисиг кошелёк

```fujin
// multisig.fjc
type @error(Error) = {
  message: string
}

type @proposeTransaction(ProposeTransaction) = {
  to: u64
  amount: u64
  proposer: u64
}

type @approveTransaction(ApproveTransaction) = {
  txId: u64
  approver: u64
}

type @transaction(Transaction) = {
  id: u64
  to: u64
  amount: u64
  approvals: u64
  executed: bool
}

type @transactionResult(TransactionResult) = {
  txId: u64
  approvals: u64
  executed: bool
}

actor @proposeTransaction(msg) {
  if (!isOwner(msg.proposer)) {
    emit @error({ message: "Not an owner" })
    return
  }
  
  const txId = createTransaction(msg.to, msg.amount)
  
  emit @transactionResult({
    txId: txId,
    approvals: 0,
    executed: false
  })
}

actor @approveTransaction(msg) {
  if (!isOwner(msg.approver)) {
    emit @error({ message: "Not an owner" })
    return
  }
  
  const tx = getTransaction(msg.txId)
  
  if (tx.executed) {
    emit @error({ message: "Already executed" })
    return
  }
  
  if (hasApproved(msg.txId, msg.approver)) {
    emit @error({ message: "Already approved" })
    return
  }
  
  const newApprovals = tx.approvals + 1
  updateApprovals(msg.txId, newApprovals)
  markAsApproved(msg.txId, msg.approver)
  
  const requiredApprovals: u64 = 3
  let executed = false
  
  if (newApprovals >= requiredApprovals) {
    executeTransaction(msg.txId)
    executed = true
  }
  
  emit @transactionResult({
    txId: msg.txId,
    approvals: newApprovals,
    executed: executed
  })
}
```

---

## Работа с деньгами без float

### Используйте минимальные единицы

```fujin
// ❌ Неправильно (float)
let price: f64 = 19.99
let total: f64 = price * 3

// ✓ Правильно (целые числа в минимальных единицах)
let priceInCents: u64 = 1999  // $19.99
let total: u64 = priceInCents * 3  // $59.97
```

### Проценты и доли

```fujin
// Проценты через умножение и деление
let amount: u64 = 1000000  // $10,000.00
let feePercent: u64 = 25   // 2.5% (умножаем на 10)
let fee: u64 = (amount * feePercent) / 1000

// Доли через basis points (1/10000)
let basisPoints: u64 = 250  // 2.5%
let fee2: u64 = (amount * basisPoints) / 10000
```

### Деление с округлением

```fujin
// Округление вниз (по умолчанию)
let result1: u64 = 100 / 3  // 33

// Округление вверх
let result2: u64 = (100 + 2) / 3  // 34

// Округление к ближайшему
let result3: u64 = (100 + 1) / 3  // 34
```

---

## Детерминированность

### Что гарантируется

В `.fjc` файлах гарантируется:
- Одинаковый результат на всех платформах
- Предсказуемое выполнение
- Консенсус в распределённых системах
- Точность финансовых операций

### Примеры детерминированности

```fujin
// ✓ Детерминированно
let a: u64 = 1000000
let b: u64 = 3
let result: u64 = a / b  // Всегда 333333

// ✓ Детерминированно
let price: u64 = 1999
let quantity: u64 = 5
let total: u64 = price * quantity  // Всегда 9995

// ❌ НЕ детерминированно (если бы был доступен float)
// let x: f64 = 0.1
// let y: f64 = 0.2
// let z: f64 = x + y  // Может быть 0.30000000000000004
```

---

## Ограничения и best practices

### Ограничения .fjc

1. **Нет float типов** — используйте целые числа в минимальных единицах
2. **Нет расширенных операций** — только базовые арифметические
3. **Нет сложных математических функций** — sqrt, pow и т.д. недоступны

### Best Practices

1. **Деньги в минимальных единицах:**
   ```fujin
   // Wei для Ethereum, satoshi для Bitcoin, копейки для рублей
   let amountInWei: u64 = 1000000000000000000  // 1 ETH
   ```

2. **Проценты через целые числа:**
   ```fujin
   // Basis points или умножение на 10/100/1000
   let feeInBasisPoints: u64 = 250  // 2.5%
   ```

3. **Избегайте переполнения:**
   ```fujin
   // Проверяйте перед операциями
  if (a > u64.max - b) {
    emit @error({ message: "Overflow" })
    return
  }
   let sum = a + b
   ```

4. **Явная обработка ошибок:**
   ```fujin
  if (balance < amount) {
    emit @error({ message: "Insufficient balance" })
    return
  }
   ```

---

## Когда использовать .fjc

### Используйте .fjc для:

- Блокчейн смартконтрактов
- Финансовых операций
- Консенсус-критичной логики
- Детерминированных вычислений
- Аудируемого кода
- Систем с требованием абсолютной точности

### НЕ используйте .fjc для:

- Графики и визуализации (нужны float)
- Научных вычислений (нужны float)
- Физических симуляций (нужны float)
- Общих бизнес-скриптов (используйте .fjs)
- UI компонентов (используйте .fjx)

---

## Миграция из .fjs в .fjc

Если вам нужно перенести логику из `.fjs` в `.fjc`:

```fujin
// Было в .fjs
let price: f64 = 19.99
let discount: f64 = 0.1
let finalPrice: f64 = price * (1.0 - discount)

// Стало в .fjc
let priceInCents: u64 = 1999        // $19.99
let discountPercent: u64 = 10       // 10%
let discountAmount: u64 = (priceInCents * discountPercent) / 100
let finalPriceInCents: u64 = priceInCents - discountAmount
```

---

## Резюме

- **`.fjc`** — файлы смартконтрактов без float
- **Детерминированность** — главная цель
- **Целочисленная арифметика** — единственный способ вычислений
- **Минимальные единицы** — для денег и процентов
- Используйте для блокчейна и критичных систем
- Для остального используйте `.fjs` с float
