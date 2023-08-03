---
title: "1 股票的识别码"
date: 2023-08-02
draft: false
description: ""
tags: 
    - stocks
    - identifier
categories: 
    - Quant-Bottom-Up
lightgallery: true
---

既然我们选择做股票的量化交易，首先我们应该了解我们交易的标的物，而不是仅仅把它们当做屏幕上的一个走势图。须知：魔鬼都在细节中。
这一小节，我们来具体了解一些股票识别码的相关知识。

标识符（Identifiers）是指用于唯一标识或识别股票的特定代码或标记。这些标识符通常由金融市场或交易所分配，以确保股票可以在交易所上正确定位和交易。常见的股票标识符包括：

- 股票代码（Stock Symbol）：由一组字母代表公司的简称，例如，AAPL代表苹果公司，GOOGL代表谷歌母公司Alphabet。
- ISIN（国际证券编码）：由12个字母和数字组成的国际标准证券编码，用于唯一标识一种证券。
- CUSIP（美国证券识别编码）：由9个字符组成的美国证券标识码。
- SEDOL（伦敦证券识别码）：由7个字符组成的伦敦证券标识码。

除了以上标识符外，一般的数据提供商还会有自己的识别符，比如：

- Bloomberg Ticker
- Reuters RIC

Bloomberg 和 Reuters 属于业内非常普遍的数据提供商，他们的标识符也被很多人认可。
于此同时，其他的数据供应商可能会有自己独家的标识符，不过通常他们也会为自己的标识符配合其他标识符的映射，比如 Bloomberg 或者 ISIN 等等。
这样，不同的数据之间才能相互连接，协调起来。

Stock Symbol 是交易所级别的标识符，而 ISIN 是股票本身级别的。

比如苹果公司的股票，其 ISIN 是 US0378331005，这只股票可以被多家交易所交易，比如 [法兰克福证券交易所](https://www.boerse-frankfurt.de/equity/apple-inc) 和 [伦敦证券交易所](https://www.londonstockexchange.com/market-stock/0R2V/apple-inc/overview).

CUSIP 主要用于美国和加拿大股票的识别，而 SEDOL 主要用于伦敦的股票识别。不过很多非地区股票也会有 CUSIP 和 SEDOL，比如苹果公司的
CUSIP 是 037833100，SEDOL 为 2046251。

Bloomberg Ticker 主要分成两种：合成和交易所。比如苹果公司：

- 合成（composite）：AAPL US Equity
- 交易所（exchange）：AAPL UW (纳斯达克交易所)

RIC 由两部分组成：Root（根）和交易所。比如

- 合成： `AAPL.O`。
- 交易所：`AAPL.OQ` 纳斯达克，`AAPL.P` ARCA

数据提供商的识别符区分交易所主要是方便考虑不同的价格信息，因为理论上不同交易所的同一个股票（ISIN）
不一定是一样的成交价格，而且订单部也相同。

当公司发生变化（Corporate Actions），比如改名、收购、Spin-off等等，这些识别符可能发生变化，需要额外注意。比如 Spin-off 可能会导致 ISIN 变化。
而作为数据的下游使用者，应该根据自己系统的情况酌情处理这种识别符变更事件，一方面是对回测的影响，另一方面是对实盘交易的影响。
