# technopark-task-2
Задание 2 к курсу "Разработчик криптографических протоколов и децентрализованных систем" Технопарка

### Задание:
просмотреть код контрактов заданного преподавателем проекта, найти место, реализующее определенную логику и привести diff, изменяющий поведение смарт-контракта:
* `MultiSigWallet.sol` - сделать, чтобы с баланса multisig-контракта за одну транзакцию не могло бы уйти больше, чем 66 ETH
* `ERC20.sol` - сделать, чтобы токен не мог быть transferred по субботам
* `DividendToken.sol` - сделать чтобы платеж в ETH принимался только специальной функцией, принимающей помимо ETH еще комментарий к платежу (bytes[32]). Простая отправка ETH в контракт запрещена

### Решение:
```
diff --git a/DividendToken.sol b/DividendToken.sol
index fed67fe..063c366 100644
--- a/DividendToken.sol
+++ b/DividendToken.sol
@@ -38,7 +38,7 @@ contract DividendToken is StandardToken, Ownable {
         }));
     }
 
-    function() external payable {
+    function deposit(bytes32 data) external payable returns (bytes32) {
         if (msg.value > 0) {
             emit Deposit(msg.sender, msg.value);
             m_totalDividends = m_totalDividends.add(msg.value);
diff --git a/ERC20.sol b/ERC20.sol
index 1e86b32..46c2052 100644
--- a/ERC20.sol
+++ b/ERC20.sol
+++ b/ERC20.sol
@@ -32,6 +32,9 @@ import "./IERC20.sol";
  * allowances. See {IERC20-approve}.
  */
 contract ERC20 is Context, IERC20 {
+       
+       uint constant DAY_IN_SECONDS = 86400;    
+
     mapping (address => uint256) private _balances;
 
     mapping (address => mapping (address => uint256)) private _allowances;
@@ -211,6 +214,10 @@ contract ERC20 is Context, IERC20 {
         require(sender != address(0), "ERC20: transfer from the zero address");
         require(recipient != address(0), "ERC20: transfer to the zero address");
 
+        current_time = now;
+        current_weekday = uint8((current_time / DAY_IN_SECONDS + 4) % 7);
+        require(current_weekday != 5, "ERC20: transfers on Saturday are forbidden")
+
         _beforeTokenTransfer(sender, recipient, amount);
 
         require(_balances[sender] >= amount, "ERC20: transfer amount exceeds balance");
diff --git a/MultiSigWallet.sol b/MultiSigWallet.sol
index 6ff26e3..27edd72 100644
--- a/MultiSigWallet.sol
+++ b/MultiSigWallet.sol
@@ -24,6 +24,7 @@ contract MultiSigWallet {
      *  Constants
      */
     uint constant public MAX_OWNER_COUNT = 50;
+    uint constant public MAX_VALUE_LIMIT = 66;
 
     /*
      *  Storage
@@ -93,6 +94,12 @@ contract MultiSigWallet {
         _;
     }
 
+    modifier isLimited(uint transactionId) {
+        require(transactions[transactionId].value <= MAX_VALUE_LIMIT,
+            "Value Limit: Unable to pass more than 66 ETH by one transaction");
+        _;
+    }
+
     /// @dev Fallback function allows to deposit ether.
     function()
         payable
@@ -202,6 +209,7 @@ contract MultiSigWallet {
         public
         ownerExists(msg.sender)
         transactionExists(transactionId)
+        isLimited(transactionId)
         notConfirmed(transactionId, msg.sender)
     {
         confirmations[transactionId][msg.sender] = true;
(END)


```

### Как делать:
Делайте форк репозитария. Изменяйте код.
Плюсом будет, если разберетесь, как дублировать репу, сделав ее приватной, а не публичный форк.
### Дублировать репу и сделать приватной:
1. `git clone --bare https://github.com/mixbytes/technopark-task-2.git`
2. `cd technopark-task-2.git/`
3. В гитхабе создайте новый приватный репозитарий и скопируйте его ссылку.
4.  `git push --mirror <ссылка на новую репу>`
5.  Старый репозитарий можно удалить:
    `cd .. && rm -rf technopark-task-2.git/`
6. Склонируйте свою новую репу:
    `git clone <ссылка на новую репу>`
7. `cd <новая репа>`

После этих шагов получайте diff 2-м способом.    

### Достать diff репозитория можно 2 способами:
1. Изменяйте код и **до коммита** выполните `git diff`, чтобы достать все изменения файлов. 
2. После коммитов можно получить все различия с форка и оригинальной репы следующим образом:
  ```bash
  git remote add original https://github.com/mixbytes/technopark-task-2.git
  git fetch original
  git diff HEAD original/main
  ```

Копируйте всю выдачу консоли, отправляйте в задание на портале.

