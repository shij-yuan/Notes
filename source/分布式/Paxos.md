# Paxos

## 角色

1. **Proposer** : `Proposer` 可以 **提出提案** (`Proposal`)。
2. **Accecptor** : `Acceptor` 可以 **接受提案**。一旦接受提案，**提案** 里面的 `value` 值就被选定了。
3. **Learner** : `Acceptor` 告诉 `Learner` 哪个提案被选定了，那么 `Learner` 就学习这个被选择的 `value`。

## 前置条件

**P：提案  V：value**

1. 在被提出的 `P` 中，只有一个 `V` 被选中。
2. 如果没有 `P` 被提出，就没有 `V` 被选中。
3. 在 `P` 被选定后，进程都可以学习被选中的 `P`。

