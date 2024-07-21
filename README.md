![Logo](./logo/logo.png)

[![Node.js CI](https://github.com/node-cache/node-cache/workflows/Node.js%20CI/badge.svg?branch=master)](https://github.com/node-cache/node-cache/actions?query=workflow%3A%22Node.js+CI%22+branch%3A%22master%22)
![Dependency status](https://img.shields.io/david/node-cache/node-cache)
[![NPM package version](https://img.shields.io/npm/v/node-cache?label=npm%20package)](https://www.npmjs.com/package/node-cache)
[![NPM monthly downloads](https://img.shields.io/npm/dm/node-cache)](https://www.npmjs.com/package/node-cache)
[![GitHub issues](https://img.shields.io/github/issues/node-cache/node-cache)](https://github.com/node-cache/node-cache/issues)
[![Coveralls Coverage](https://img.shields.io/coveralls/node-cache/node-cache.svg)](https://coveralls.io/github/node-cache/node-cache)

# 簡單且快速的 NodeJS 內部快取

一個簡單的快取模組，具有 `set`、`get` 和 `delete` 方法，工作方式有點像 memcached。
鍵可以設置一個超時時間（`ttl`），在該時間後過期並從快取中刪除。
所有的鍵都存儲在一個單一的對象中，所以實際的限制大約是 1 百萬個鍵。

## 宣言不會出售

我們過去收到過許多請求，要求將這個專案交給新的維護者，以便它可以復甦並重獲生機。
許多這些請求還提供了金錢，實質上是“購買”這個專案（或其聲譽和用戶基礎）。

這些請求對於當前使用這個專案的用戶來說是非常風險的，尤其是在 xz-backdoor 時期，並且將用戶基礎（主要由 [@mpneuried](https://github.com/mpneuried) 和 [Team Centric Software](https://www.tcs.de/) 贊助）建立起來的信任交給他人感覺就像是對用戶的不尊重。
將這個專案的密鑰（尤其是發佈新版本的 npm 包的權限）交給新的人包括將我們用戶對這個包的信任一同交出。

因此 **我們發誓不會出售這個專案及其每週 2,886,264 次下載量（根據 2024 年 5 月 29 日的 npm 統計數據）**。

是的，這個專案顯然未得到維護。如果你想補上這個缺口，可以隨時創建一個 fork 並開始在那裡維護。如果社群需要，它會跟隨。

## 重大的破壞性版本發佈 v5.x

最近的 5.x 版本發佈：
* 不再支持 8.x 之前的 Node 版本！
* 從所有方法中移除了基於回調的 API（可以通過選項 `enableLegacyCallbacks` 重新啟用它們）

## 即將到來的重大破壞性版本發佈 v6.x

雖然根據定義不是破壞性的，但我們的 TypeScript 重寫將改變內部函數及其名稱。
如果你正在使用 node-cache 的某些內部 API，請與我們聯繫，以便我們可以一起解決這個問題！

# 安裝

```bash
	npm install node-cache --save
```

或者只需 require `node_cache.js` 文件來獲取超類。

# 範例：

## 初始化 (INIT)：

```js
const NodeCache = require("node-cache");
const myCache = new NodeCache();
```

### 選項

- `stdTTL`: *(預設值: `0`)* 每個生成的快取元素的標準 TTL（以秒為單位）。`0` = 無限制。
- `checkperiod`: *(預設值: `600`)* 用於自動刪除檢查間隔的時間（以秒為單位）。`0` = 無定期檢查。
- `useClones`: *(預設值: `true`)* 啟用/禁用變量的克隆。如果 `true`，你將獲得快取變量的副本。如果 `false`，你將只儲存和獲取引用。
  **注意：**
  - 如果你想要**簡單性**，建議設為 `true`，因為它將像基於伺服器的快取一樣（快取純數據的副本）。
  - 如果你想要**性能**或保存可變對象或其他涉及可變性的複雜類型，並且希望這些類型是可變的，建議設為 `false`，因為它只會儲存數據的引用。
  - 這裡有一個[簡單的代碼範例](https://runkit.com/mpneuried/useclones-example-83)顯示了不同的行為。
- `deleteOnExpire`: *(預設值: `true`)* 變量過期時是否會自動刪除。如果 `true`，變量將被刪除。如果 `false`，變量將保留。建議你自己處理 `expired` 事件。
- `enableLegacyCallbacks`: *(預設值: `false`)* 重新啟用回調而不是同步函數的使用。為每個函數添加一個額外的 `cb` 參數，它解析為 `(err, result)`。在 node-cache v6.x 中將被移除。
- `maxKeys`: *(預設值: `-1`)* 指定可以儲存在快取中的鍵的最大數量。如果快取已滿並設置新項目，將引發錯誤，並且該鍵將不會保存到快取中。`-1` 禁用鍵限制。

```js
const NodeCache = require("node-cache");
const myCache = new NodeCache({ stdTTL: 100, checkperiod: 120 });
```

**自 `4.1.0` 起**：
*鍵驗證*：鍵可以是 `string` 或 `number`，但會內部轉換為 `string`。
所有其他類型將引發錯誤。

## 儲存一個鍵 (SET)：

`myCache.set( key, val, [ ttl ] )`

設置 `key` `value` 對。可以定義 `ttl`（以秒為單位）。
成功時返回 `true`。

```js
obj = { my: "Special", variable: 42 };

success = myCache.set("myKey", obj, 10000);
// true
```

> 注意：如果鍵基於其 `ttl` 過期，將完全從內部數據對象中刪除。

## 儲存多個鍵 (MSET)：

`myCache.mset(Array<{key, val, ttl?}>)`

設置多個 `key` `val` 對。可以定義 `ttl`（秒）。
成功時返回 `true`。

```js
const obj = { my: "Special", variable: 42 };
const obj2 = { my: "other special", variable: 1337 };

const success = myCache.mset([
	{key: "myKey", val: obj, ttl: 10000},
	{key: "myKey2", val: obj2},
]);
```

## 檢索一個鍵 (GET)：

`myCache.get( key )`

從快取中獲取已保存的值。
如果未找到或已過期，返回 `undefined`。
如果找到值，則返回 `value`。

```js
value = myCache.get("myKey");
if (value == undefined) {
	// handle miss!
}
// { my: "Special", variable: 42 }
```

**自 `2.0.0` 起**：

返回格式更改為簡單的值和 `ENOTFOUND` 錯誤，如果未找到（作為 `Error` 的實例結果）

**自 `2.1.0` 起**：

返回格式更改為簡單的值，但由於討論在 #11 中，未命中不應返回錯誤。
所以自 2.1.0 起，未命中返回 `undefined`。

## 獲取並刪除一個鍵 (TAKE)：

`myCache.take( key )`

獲取快取值並從快取中刪除鍵。
相當於調用 `get(key)` + `del(key)`。
用於實現 `單次使用` 機制，例如 OTP，一旦讀取值將變得過時。

```js
myCache.set("myKey", "myValue");
myCache.has("myKey"); // returns true because the key is cached right now
value = myCache.take("myKey"); // value === "myValue"; this also deletes the key
myCache.has("myKey"); // returns false because the key has been deleted
```

## 獲取多個鍵 (MGET)：

`myCache.mget( [ key1, key2, ..., keyn ] )`

從快取中獲取多個已保存的值。
如果未找到或已過期，返回一個空對象 `{}`。
如果找到值，則返回帶有 `key` `value` 對的對象。

```js
value = myCache.mget(["myKeyA", "myKeyB"]);
/*
	{
		"myKeyA": { my: "Special", variable: 123 },
		"myKeyB": { the: "Glory", answer: 42 }
	}
*/
```

**自 `2.0.0` 起**：

`mget` 方法從 `.get( [ "a", "b" ] )` 更改為 `.mget( [ "a", "b"

 ] )`

## 刪除一個鍵 (DEL)：

`myCache.del( key )`

刪除一個鍵。返回刪除的條目數。刪除永遠不會失敗。

```js
value = myCache.del("A");
// 1
```

## 刪除多個鍵 (MDEL)：

`myCache.del( [ key1, key2, ..., keyn ] )`

刪除多個鍵。返回刪除的條目數。刪除永遠不會失敗。

```js
value = myCache.del("A");
// 1

value = myCache.del(["B", "C"]);
// 2

value = myCache.del(["A", "B", "C", "D"]);
// 1 - 因為 A、B 和 C 不存在
```

## 更改 TTL (TTL)：

`myCache.ttl( key, ttl )`

重新定義鍵的 TTL。如果找到並更改鍵，則返回 true。否則返回 false。
如果沒有傳遞 ttl 參數，將使用預設 TTL。

傳入 `ttl < 0` 時，鍵將被刪除。

```js
myCache = new NodeCache({ stdTTL: 100 });
changed = myCache.ttl("existentKey", 100);
// true

changed2 = myCache.ttl("missingKey", 100);
// false

changed3 = myCache.ttl("existentKey");
// true
```

## 獲取 TTL (getTTL)：

`myCache.getTtl( key )`

獲取鍵的 TTL。
你將獲得：
- `undefined` 如果鍵不存在
- `0` 如果該鍵沒有 ttl
- 以毫秒為單位的時間戳，表示鍵將過期的時間

```js
myCache = new NodeCache({ stdTTL: 100 });

// Date.now() = 1456000500000
myCache.set("ttlKey", "MyExpireData");
myCache.set("noTtlKey", "NonExpireData", 0);

ts = myCache.getTtl("ttlKey");
// ts 大約為 1456000600000

ts = myCache.getTtl("ttlKey");
// ts 大約為 1456000600000

ts = myCache.getTtl("noTtlKey");
// ts = 0

ts = myCache.getTtl("unknownKey");
// ts = undefined
```

## 列出所有鍵 (KEYS)

`myCache.keys()`

返回所有存在的鍵的數組。

```js
mykeys = myCache.keys();

console.log(mykeys);
// [ "all", "my", "keys", "foo", "bar" ]
```

## 檢查鍵是否存在 (HAS)

`myCache.has( key )`

返回表示鍵是否快取的布林值。

```js
exists = myCache.has('myKey');

console.log(exists);
```

## 獲取統計信息 (STATS)：

`myCache.getStats()`

返回統計信息。

```js
myCache.getStats();
/*
	{
		keys: 0,    // 全局鍵數量
		hits: 0,    // 全局命中次數
		misses: 0,  // 全局未命中次數
		ksize: 0,   // 全局鍵大小（近似字節）
		vsize: 0    // 全局值大小（近似字節）
	}
*/
```

## 清空所有數據 (FLUSH)：

`myCache.flushAll()`

清空所有數據。

```js
myCache.flushAll();
myCache.getStats();
/*
	{
		keys: 0,    // 全局鍵數量
		hits: 0,    // 全局命中次數
		misses: 0,  // 全局未命中次數
		ksize: 0,   // 全局鍵大小（近似字節）
		vsize: 0    // 全局值大小（近似字節）
	}
*/
```

## 清空統計信息 (FLUSH STATS)：

`myCache.flushStats()`

清空統計信息。

```js
myCache.flushStats();
myCache.getStats();
/*
	{
		keys: 0,    // 全局鍵數量
		hits: 0,    // 全局命中次數
		misses: 0,  // 全局未命中次數
		ksize: 0,   // 全局鍵大小（近似字節）
		vsize: 0    // 全局值大小（近似字節）
	}
*/
```

## 關閉快取：

`myCache.close()`

這將清除設置在檢查期間選項上的間隔超時。

```js
myCache.close();
```

# 事件

## set

當鍵已添加或更改時觸發。
你將獲得 `key` 和 `value` 作為回調參數。

```js
myCache.on("set", function(key, value) {
	// ... 做些什麼 ...
});
```

## del

當鍵被手動刪除或因過期而刪除時觸發。
你將獲得 `key` 和被刪除的 `value` 作為回調參數。

```js
myCache.on("del", function(key, value) {
	// ... 做些什麼 ...
});
```

## expired

當鍵過期時觸發。
你將獲得 `key` 和 `value` 作為回調參數。

```js
myCache.on("expired", function(key, value) {
	// ... 做些什麼 ...
});
```

## flush

當快取已被清空時觸發。

```js
myCache.on("flush", function() {
	// ... 做些什麼 ...
});
```

## flush_stats

當快取統計信息被清空時觸發。

```js
myCache.on("flush_stats", function() {
	// ... 做些什麼 ...
});
```

## 重大更改

### 版本 `2.x`

由於 [Issue #11](https://github.com/mpneuried/nodecache/issues/11)，`.get()` 方法的返回格式已更改！

不再返回 `{ "myKey": "myValue" }` 對象，而是返回值 `"myValue"`。

### 版本 `3.x`

由於 [Issue #30](https://github.com/mpneuried/nodecache/issues/30) 和 [Issue #27](https://github.com/mpneuried/nodecache/issues/27)，變量現在將被克隆。
這可能會破壞你的代碼，因為對於某些變量類型（例如 Promise），無法克隆它們。
你可以通過設置選項 `useClones:false` 禁用克隆。在這種情況下，它與 `2.x` 版本兼容。

### 版本 `5.x`

回調在此版本中已被棄用。啟用 `enableLegacyCallbacks` 選項時仍可使用回調。在 `6.x` 版本中將完全移除回調。

## 兼容性

Node-Cache 支持所有 >= 8 的 Node 版本。

## 發佈歷史

|版本|日期|描述|
|:--:|:--:|:--|
|5.1.2|2020-07-01|[#195] .take() 的類型定義和拼寫錯誤修正，感謝 [shhadi](https://github.com/shhadi)！, [#198]/[#197] 設置值時在沒有 `Buffer` 的 js 環境中發生錯誤，感謝 [jdussouillez](https://github.com/jdussouillez) 和 [Sirz3chs](https://github.com/Sirz3chs) 的幫助|
|5.1.1|2020-06-06|[#184], [#183] 感謝 [Jonah Werre](https://github.com/jwerre) 報告 [#181]！, [#180], 感謝 [Titus](https://github.com/tstone) 對 [#169] 的貢獻！, 感謝 [Ianfeather](https://github.com/Ianfeather) 對 [#168] 的貢獻！, 感謝 [Adam Haglund](https://github.com/BeeeQueue) 對 [#176] 的貢獻|
|5.1.0|2019-12-08|新增 .take() 自 PR [#160] 和 .flushStats 自 PR [#161]。感謝 [Sujesh Thekkepatt](https://github.com/sujeshthekkepatt) 和 [Gopalakrishna Palem](https://github.com/KrishnaPG)！|
|5.0.2|2019-11-17|修正當 `deleteOnExpire` 設置為 `false` 時過期值仍被刪除的錯誤。感謝 [fielding-wilson](https://github.com/fielding-wilson)！|
|5.0.1|2019-10-31|修正用戶無法設

置 null 值的錯誤。感謝 [StefanoSega](https://github.com/StefanoSega), [jwest23](https://github.com/jwest23) 和 [marudor](https://github.com/marudor)！|
|5.0.0|2019-10-23|移除 lodash 依賴，添加 .has(key) 和 .mset([{key,val,ttl}]) 方法到快取。感謝 [Regev Brody](https://github.com/regevbr) 提供 PR [#132] 和 [Sujesh Thekkepatt](https://github.com/sujeshthekkepatt) 提供 PR [#142]！也感謝所有未在此處命名的其他貢獻者！|
|4.2.1|2019-07-22|升級 lodash 至版本 4.17.15 以抑制有關不相關安全漏洞的消息|
|4.2.0|2018-02-01|添加 options.promiseValueSize 用於 Promise 值。感謝 [Ryan Roemer](https://github.com/ryan-roemer) 提供 pull [#84]；添加選項 `deleteOnExpire`；添加 DefinitelyTyped TypeScript 定義。感謝 [Ulf Seltmann](https://github.com/useltmann) 提供 pulls [#90] 和 [#92]；感謝 [Daniel Jin](https://github.com/danieljin) 提供 pull [#93] 中的 readme 修正；優化測試和 CI 配置。|
|4.1.1|2016-12-21|修正內部檢查間隔以支持 node < 0.10.25，這是 ubuntu 14.04 的預設 node 版本。感謝 [Jimmy Hwang](https://github.com/JimmyHwang) 提供 pull [#78](https://github.com/mpneuried/nodecache/pull/78)；添加更多 docker 測試。|
|4.1.0|2016-09-23|添加不同鍵類型的測試；添加鍵驗證（必須是 `string` 或 `number`）；修正 .del 錯誤，嘗試刪除 `number` 鍵導致無法刪除。|
|4.0.0|2016-09-20|更新測試至 mocha；修正 .ttl 錯誤不刪除鍵在 .ttl( key, 0 ) 時。如果 `stdTTL=0`，這也是相關的。*這導致 `4.0.0` 的破壞性變更*。|
|3.2.1|2016-03-21|升級 lodash 至 4.x.；優化 grunt。|
|3.2.0|2016-01-29|添加方法 `getTtl` 以獲取鍵過期時間。請參閱 [#49](https://github.com/mpneuried/nodecache/issues/49)。|
|3.1.0|2016-01-29|添加選項 `errorOnMissing` 在未命中時引發/回調錯誤。感謝 [David Godfrey](https://github.com/david-byng) 提供 pull [#45](https://github.com/mpneuried/nodecache/pull/45)。添加 docker 文件和腳本以在本地運行不同 Node 版本的測試。|
|3.0.1|2016-01-13|添加 `.unref()` 到 checkTimeout，這樣在 node `0.10` 之前不需要在腳本完成時調用 `.close()`。感謝 [Doug Moscrop](https://github.com/dougmoscrop) 提供 pull [#44](https://github.com/mpneuried/nodecache/pull/44)。|
|3.0.0|2015-05-29|返回已快取元素的克隆版本並保存變量的克隆版本。可以通過設置選項 `useClones:false` 禁用克隆。（感謝 #27 提供者 [cheshirecatalyst](https://github.com/cheshirecatalyst) 和 #30 提供者 [Matthieu Sieben](https://github.com/matthieusieben)）。|
|~~2.2.0~~|~~2015-05-27~~|撤銷版本，因為存在衝突。請參閱 [Issue #30](https://github.com/mpneuried/nodecache/issues/30)。所以 `2.2.0` 現在是 `3.0.0`。|
|2.1.1|2015-04-17|將舊值傳遞給 `del` 事件。感謝 [Qix](https://github.com/qix) 提供 pull。|
|2.1.0|2015-04-17|更改未命中返回 `undefined` 而不是錯誤。感謝所有 [#11](https://github.com/mpneuried/nodecache/issues/11) 的貢獻者。|
|2.0.1|2015-04-17|添加 close 函數（感謝 [ownagedj](https://github.com/ownagedj)）。將開發環境更改為使用 grunt。|
|2.0.0|2015-01-05|更改 `.get()` 的返回格式為錯誤返回在未命中時並添加 `.mget()` 方法。*副作用：性能提升至 330 倍更快！*。|
|1.1.0|2015-01-05|添加 `.keys()` 方法以列出所有現有鍵。|
|1.0.3|2014-11-07|修正設置數字值的錯誤。感謝 [kaspars](https://github.com/kaspars) + 優化鍵檢查。|
|1.0.2|2014-09-17|更改以更好地處理 ttl。|
|1.0.1|2014-05-22|Readme 拼寫錯誤。感謝 [mjschranz](https://github.com/mjschranz)。|
|1.0.0|2014-04-09|使 `callback` 可選。因此現在可以使用同步語法。舊語法也應該能夠正常工作。Push: 錯誤修正值 `0`。|
|0.4.1|2013-10-02|將值添加到 `expired` 事件。|
|0.4.0|2013-10-02|添加 nodecache 事件。|
|0.3.2|2012-05-31|添加 Travis 測試。|

[![NPM](https://nodei.co/npm-dl/node-cache.png?months=6)](https://nodei.co/npm/node-cache/)

## 其他專案

|名稱|描述|
|:--|:--|
|[**rsmq**](https://github.com/smrchy/rsmq)|基於 redis 的非常簡單的消息隊列|
|[**redis-heartbeat**](https://github.com/mpneuried/redis-heartbeat)|向 redis 發送心跳。這可以用於脫離或附加伺服器到 nginx 或類似問題。|
|[**systemhealth**](https://github.com/mpneuried/systemhealth)|Node 模組來運行簡單的自定義檢查你的機器或其連接。它將使用 [redis-heartbeat](https://github.com/mpneuried/redis-heartbeat) 將當前狀態發送到 redis。|
|[**rsmq-cli**](https://github.com/mpneuried/rsmq-cli)|rsmq 的終端客戶端|
|[**rest-rsmq**](https://github.com/smrchy/rest-rsmq)|的 REST 接口。|
|[**redis-sessions**](https://github.com/smrchy/redis-sessions)|NodeJS 和 Redis 的高級會話存儲|
|[**connect-redis-sessions**](https://github.com/mpneuried/connect-redis-sessions)|一個連接或 express 中間件，簡單地使用 [redis sessions](https://github.com/smrchy/redis-sessions)。使用 [redis sessions](https://github.com/smrchy/redis-sessions)，你可以處理每個 user_id 的多個會話。|
|[**redis-notifications**](https://github.com/mpneuried/redis-notifications)|基於 redis 的通知引擎。它實現了 rsmq-worker 以安全地創建通知和定期報告。|
|[**nsq-logger**](https://github.com/mpneuried/nsq-logger)|Nsq 服務，用於從所有 nsqlookupd 服務中列出的主題中讀取消息。|
|[**nsq-topics**](https://github.com/mpneuried/nsq-topics)|Nsq 幫助程序，輪詢 nsqlookupd 服務的所有主題並在本地鏡像它。|
|[**nsq-nodes**](https://github.com/mpneuried/nsq-nodes)|

Nsq 幫助程序，輪詢 nsqlookupd 服務的所有節點並在本地鏡像它。|
|[**nsq-watch**](https://github.com/mpneuried/nsq-watch)|監視一個或多個主題是否有未處理的消息。|
|[**hyperrequest**](https://github.com/mpneuried/hyperrequest)|[hyperquest](https://github.com/substack/hyperquest) 的包裝器來處理結果。|
|[**task-queue-worker**](https://github.com/smrchy/task-queue-worker)|一個強大的工具，用於後臺處理通過標準 http 請求運行的任務。|
|[**soyer**](https://github.com/mpneuried/soyer)|Soyer 是一個小型庫，用於在 node.js 中使用 Google Closure Templates。|
|[**grunt-soy-compile**](https://github.com/mpneuried/grunt-soy-compile)|編譯 Google Closure Templates (SOY) 模板，包括 XLIFF 語言文件的處理。|
|[**backlunr**](https://github.com/mpneuried/backlunr)|一個將 Backbone Collections 和瀏覽器全文搜索引擎 Lunr.js 結合的解決方案。|
|[**domel**](https://github.com/mpneuried/domel)|一個簡單的 dom 幫助程序，如果你想擺脫 jQuery。|
|[**obj-schema**](https://github.com/mpneuried/obj-schema)|簡單的模組，用於通過預定義的模式驗證對象。|

# MIT 許可證 (MIT)

版權 © 2019 Mathias Peter 及 node-cache 維護者, https://github.com/node-cache/node-cache

特此授予，免費地，獲得本軟件及相關文檔文件（“軟件”）副本的任何人以處理軟件的權利，包括但不限於使用、複製、修改、合併、發布、分發、再許可和/或銷售軟件副本的權利，並允許軟件提供者這樣做，條件是上述版權聲明和本許可聲明應包含在軟件的所有副本或主要部分中。

軟件按“原樣”提供，不提供任何形式的明示或暗示擔保，包括但不限於對適銷性、適合特定用途和非侵權性的擔保。在任何情況下，作者或版權持有人均不對因使用軟件或與軟件的使用或其他交易而引起的任何索賠、損害或其他責任承擔任何責任，無論是合同行為、侵權行為還是其他方式。
