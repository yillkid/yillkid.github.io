---
layout: post
title:  "IOTA 2017-09-12 遭受攻擊摘要"
date:   2017-09-20 01:21:02 +0800
categories: IOTA
---
# 概述
[IOTA](https://iota.org/) 為基於有向無環圖Directed Acyclic Graph（DAG）打造的第三代加密貨幣，其實現的機制稱為 Tangle。2017-09-12，IOTA 基金會的創辦人 David Sønstebø 在 [IOTA Slack](https://slack.iota.org/) 的 announcements channel 上發佈 Tangle 遭受到攻擊，為了安全起見，希望個用戶及交易所暫停交易。

```
David Sønstebø [12:33 AM] 
@here Someone is attempting to attack the mainnet at the moment, for security precautionary sake we encourage you to not do value transfers at the moment, while we analyze the details of this attempted attack. Bitfinex and YDX has been informed as well.
```
![](https://i.imgur.com/PWiubWq.png)
```
David Sønstebø [4:20 AM] 
@here Thus far it seems indeed that there was never any actual risk involved here. We are still sorting through the details and will release a patch and details shortly. These kinds of attacks provide us with a lot of free data to conduct further research on how Tangle operates under unique circumstances
```
![](https://i.imgur.com/PcoD8cr.png)
```
David Sønstebø [6:20 AM] 
@here a blog post will come tomorrow which elucidates everything and provide the patch. What we can say already is that this 'attack' has validated IOTA further
```
![](https://i.imgur.com/g3S5oXL.png)

2017-09-14 David Sønstebø 在 medium 上發佈了名為 [Simple bug, simple fix. Network resilience verified.](https://blog.iota.org/simple-bug-simple-fix-network-resilience-verified-162161acae77) 的摘要。根據 David 的解釋，Tangle 的運作是所有節點基於 Coordinator 機制的實現，因此單一節點的運作並不重要，因此攻擊者便利用節點們的 bug 進行攻擊，也造成一些節點的錯誤，IOTA 社群最後決定發布一個新的 [release](https://github.com/iotaledger/iri/releases/tag/v1.3.2.2) 修正這個問題。（David 並非技術人員，所以他的描述不會交待太多技術細節，因此本次攻擊技術層面的詳細解說請慢慢看下去。）

由於攻擊的時間點接近 IOTA 社群內的開發人員 [Peter Ryszkiewicz](https://github.com/pRizz) 發布一套 [SPAMMER](https://github.com/pRizz/iota-transaction-spammer-webapp) 的時間，一度還引發懷疑是否與其相關。但其實 IOTA 本質是可以抵禦 SPAM 攻擊的，甚至 SPAM 還能夠促進 Tangle 交易的速度，所以這個懷疑論很快就得到澄清。Peter Ryszkiewicz 也將這個澄清公佈在專案頁面上。
```
Warning
×
As of September 11, 2017 (Never Forget)
This webapp has been blacklisted by the https node operators due to network/performance issues. Running this webapp over GitHub Pages forces an https connection, so this webapp must connect to https nodes. But since there were only two https nodes in the main list, they were getting hammered by traffic. I have also disabled the https nodes from being spammed at the request of their owners.
One solution to this is to download this webapp and run it locally. This will allow you to hit the http-only nodes and spread out the traffic. https://github.com/pRizz/iota-transaction-spammer-webapp/releases/download/1.0/iota-transaction-spammer-webapp.zip Just download the zip, unzip, and open the index.html file in your browser.
About the network attacks
This tool was not made with the intention of hurting the IOTA network, but rather strengthening it. IOTA is still very early in its development and will need to handle traffic orders of magnitude higher than the people spamming the network today, in order to succeed as a cryptocurrency in the future. This "attack" acts as a test for the network that can be utilized to further improve the code, fix bugs and performance issues. Don't panic just because of one little bump in the road. IOTA has a long and exciting future ahead!
```
![](https://i.imgur.com/zx5Ppa5.png)

終於在當日 (2017-09-12) 晚上 IOTA 發布了一項[修正](https://github.com/iotaledger/iri/commit/1dfbbd98081c0e95c25f325e3cc9c34fe5a4edf7#diff-8191e395a49bc186f7c1c7c6ebfb723f)

IOTA 的資深工程師 Come_from_Beyond 也分別在 Reddit 和 Slack 上稍做解釋：
Reddit 上的[解釋](https://www.reddit.com/r/Iota/comments/6zwnzb/simple_bug_simple_fix_network_resilience_verified/dmymh1i/)：
```
A Coordinator milestone is a bundle consisting of 2 transactions. The first transaction contains a tag equal to the milestone index and the signature. The second transaction contains service data required for the signature verification. The attacker exploited the fact that during one of the optimizations the check for a milestone index being within some range was removed. He generated another transaction being a 90% copy of the first one but with a tag with a higher milestone index. That is a pretty standard bug.
```

Slack 上的解釋：
```
The nature of the attack was the following: Take legit milestone and increase its index by 0x200000. Next time Coo issues next milestone it won't be accepted coz its index is lower.
```

# 討論
#### Coordinator and Milestone
---
IOTA 目前正處於大規模部署和標準化的過渡期之中，Tangle 需要一個入門機制，以便在早期提供 34％ 的攻擊保護，稱之為 Coordinator，直到分佈式賬本的活動的數量足以使其在沒有保護協助的情況下發展，這時就會永久關閉。而 Coordinator 本質是一個修改過得 node，以一組固定的 [Address(KPWCHICGJZXKE9GSUDXZYUAPLHAKAHYHDXNPHENTERYMMBQOPSQIDENXKLKCEYCPVTZQLEEJVYJZV9BWU)](https://github.com/iotaledger/iri/blob/4985b4c4a55891462a89a51dc18fb50be6d7ae01/src/main/java/com/iota/iri/Iota.java#L32) 不斷地確認 Tangle 上尚未被 confirm 的交易（尚未被 confirm 的交易稱之為 tips）。每確認一批 tips 之後，Coordinator 便會發起一束（bundle）特殊的交易，此束交易稱之為 Milestone，我們可以從 Tangle Explorer 上查得所有的 [Milestones](https://tangler.herokuapp.com/search/?kind=address&hash=KPWCHICGJZXKE9GSUDXZYUAPLHAKAHYHDXNPHENTERYMMBQOPSQIDENXKLKCEYCPVTZQLEEJVYJZV9BWU)（如果頁面遇到 connection refused 錯誤，多 reload 幾次就會成功了）。

而一束 Milestone 又可以拆開成為兩筆交易 (transaction)，這兩筆交易以 tags 的資料來描述該 Milestone，兩筆交易的 tag 的資料如下：

第一筆交易上的 tag 紀錄著兩筆資訊：
1. Coordinator 目前在 Tangle 上確認過的交易進度，為一個正整數值，稱之為 MilestoneIndex.
2. 用來驗證此 Milestone 的簽名 (signature)。 

第二筆交易的 tag ：
1. 紀錄著用來驗證 Milestone 的相關資訊。

IOTA node (IRI) 在啟動時，會直接搜尋 Coordinator Address 簽署過的交易，也就是濾出所有的 Milstone，並且以 MilestoneIndex 最高的數目的交易為基準點，以此數值開始進行與 Coordinator 的分散式帳本同步，詳細的程式碼如下 [detail](https://github.com/iotaledger/iri/blob/878fd31320540587825c774c96a37ab756f33dea/src/main/java/com/iota/iri/Milestone.java#L84)：

```
try {
    final int previousLatestMilestoneIndex = latestMilestoneIndex;
    Set<Hash> hashes = AddressViewModel.load(tangle, coordinator).getHashes();
    { // Update Milestone
        { // find new milestones
            for(Hash hash: hashes) {
                if(analyzedMilestoneCandidates.add(hash)) {
                    TransactionViewModel t = TransactionViewModel.fromHash(tangle, hash);
                    if (t.getCurrentIndex() == 0) {
                        if (validateMilestone(mode, t, getIndex(t))) {
                            MilestoneViewModel milestoneViewModel = MilestoneViewModel.latest(tangle);
                            if (milestoneViewModel != null && milestoneViewModel.index() > latestMilestoneIndex) {
                                latestMilestone = milestoneViewModel.getHash();
                                latestMilestoneIndex = milestoneViewModel.index();
                            }
                        } else {
                            analyzedMilestoneCandidates.remove(t.getHash());
                        }
                    }
                }
            }
        }
    }
```

#### Attack
---
由上述的程式碼可以得知，IRI 的運作機制只是單純的將 Coordinator 簽署過的 Milestone 讀進來作為判斷基準而已，並未對讀進來的資料做太多判斷。攻擊者便利用這一項弱點，自行複製了一模一樣的 Milestone，方法為直接 copy 一筆舊 Milestone 交易的 raw data（為 trytes 格式），唯一不同的地方在於：

1. 攻擊者竄改了 tag 上的 MilestoneIndex，將其值設定為比 Tangle 上的其他 Milestone 都還要高，因為 IRI 的機制為以 MilestoneIndex 最高值為基準點，所以這筆交易會誤導 Tangle 上的節點，使其無法與真正的網路同步，這會造成節點上的某些需要 fully-synced 的 API command 無法正常運作。
2. 即便攻擊者可以複製一個一模一樣的交易，但這項簽署肯定是無效的，因為攻擊者並沒有取得真正的 private key，但 IRI 並沒有確認這項簽署的機制，故 IRI 也無法判斷真偽。

被誤導的節點，因為接收了錯誤的 MileStoneIndex，基準點也全部都處於錯誤的狀態，而且完全無法與網路同步（因為 Coordinator 並沒有真正確認這些交易）。我們可以透過 Restful API command **NodeInfo** 來查看 MilestoneIndex 被竄改過後的情況：

![](https://i.imgur.com/oYBZ71L.png)

```
Every 2.0s: ./tools/node_info.sh                                                                                                         node0: Fri Sep 15 00:10:21 2017

{
    "appName": "IRI",
    "appVersion": "1.3.2.2",
    "duration": 0,
    "jreAvailableProcessors": 24,
    "jreFreeMemory": 148599936,
    "jreMaxMemory": 894959616,
    "jreTotalMemory": 670040064,
    "jreVersion": "1.8.0_144",
    "latestMilestone": "I9QBZNN9QJPEDUGKRKEXHQVTRQLIGVNFGJCIJFGWVAYZJWSTQYFWSPFEMBYFACGQTKAQTTWFIYLN99999",
    "latestMilestoneIndex": 6492449,
    "latestSolidSubtangleMilestone": "PHWWPC9BHZGDWROBRMV9OUCO9XQJRHCRIXHHLDUXOQIRZLNCBWQECECSPWGBUKAPVHRHMKMJGVQC99999",
    "latestSolidSubtangleMilestoneIndex": 205137,
    "neighbors": 11,
    "packetsQueueSize": 0,
    "time": 1505405421756,
    "tips": 309,
    "transactionsToRequest": 1
}
```

IOTA 資深工程師 Come_from_Beyond 表示這並非是 IOTA Tangle 演算法上的缺失，而是一個標準的 bug (That is a pretty standard bug). 這個說法的立基點在於，攻擊者其實仍無法進行雙重支付（double spending）、竄改 Tangle 上的交易，或是企圖 hard fork Tangle，本次攻擊造成最大的影響就是讓節點誤判網路同步的情況，最大的影響是造成某些 API command 無法使用（有些 API command 需要在 Tangle funny-synced 之下運行），但即便是 MilestoneIndex 被竄改，但因為 Tangle 上目前只有 Coordinator 有 confirm 的權力，所以 Tangle 上交易確認機制不會受到影響。

#### Patch
---
當日晚上 IOTA 社群即釋出一段 [patch](https://github.com/iotaledger/iri/commit/1dfbbd98081c0e95c25f325e3cc9c34fe5a4edf7#diff-8191e395a49bc186f7c1c7c6ebfb723f) 來對漏洞進行修補，修補的方向為：

1. 在 Mailestone 確認機制啟動時，確認 MilestoneIndex 的上下限。
  * MilestoneIndex 下限需為正值。
  * MilestoneIndex 上限須為 0x200000，這個數值為 Coordinator 的退場機制。
2. 檢查 IRI 讀取的 Meilstone 內含的 transactions 是否均為同一個 bundle hash
3. 在利用 Merkle Tree 驗證 Coordinator 確認過得 transactions 時，確認其數量是否與 MilestoneIndex 相同。 

# Reference
* [Simple bug, simple fix. Network resilience verified. (David Sønstebø post on medium)](https://blog.iota.org/simple-bug-simple-fix-network-resilience-verified-162161acae77)
* [IRI 1.3.2.2](https://github.com/iotaledger/iri/releases/tag/v1.3.2.2)
* [Simple bug, simple fix. Network resilience verified. (discuss in reddit)](https://www.reddit.com/r/Iota/comments/6zwnzb/simple_bug_simple_fix_network_resilience_verified/dmymh1i/)
