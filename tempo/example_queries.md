# Background

Tempo is a blockchain designed and built for real-world payments:
* Learn more about Tempo [here](https://docs.tempo.xyz/)
* Tempo's first public testnet is called **Andantino**

Tempo Andantino (Testnet) data is now available for use via Allium App, Allium Datashares and Allium Datastreams 🎉

# Example Queries

## Example 1: Daily transaction count
```
select
    date(timestamp) as date,
    sum(transaction_count) as num_transactions
from tempo_andantino.raw.blocks
group by all
order by 1 desc;
```

## Example 2: Median block times

Tempo targets ~0.5 second block times

```
with
    block_times as (
        select
            timestamp,
            number,
            timestamp_millis,
            lead(timestamp_millis) over (order by timestamp_millis desc) as prev_timestamp,
            (timestamp_millis - prev_timestamp) / pow(10,3) as block_time_seconds,
        from tempo_andantino.raw.blocks
    )

    select 
        date(timestamp),
        median(block_time_seconds) as median_block_time_seconds,
    from block_times
    group by all
    order by 1 desc
```

## Example 3: Daily fee token usage for transactions

Tempo doesn’t use a native gas token. Instead, transaction fees are set in USD terms and can be paid using a stablecoin

* For [Tempo Transactions](https://docs.tempo.xyz/protocol/transactions), the fee token can be set to any [TIP-20 token](https://docs.tempo.xyz/protocol/tip20/overview)
* For all other transactions, there is a fee token selection algorithm based on the user's preferences and the contracts being called
* [pathUSD](https://docs.tempo.xyz/protocol/exchange/pathUSD) with address `0x20c0000000000000000000000000000000000000` is used as a fallback gas token

```
select
    date(block_timestamp) as date,
    nvl(receipt_fee_token, '0x20c0000000000000000000000000000000000000') as receipt_fee_token,
    count(*)
from tempo_andantino.raw.transactions
group by all
order by 1 desc;
```

## Example 4: Fetch specific transaction details

```
select
    *
from tempo_andantino.raw.transactions
where 1=1
    and hash = <TRANSACTION_HASH>;
```

## Example 5: Fetch event logs for a given transaction

Logs (or event logs), are a way for smart contracts to communicate with the outside world by “documenting” small pieces of information.

```
select
    *
from tempo_andantino.raw.logs
where 1=1
    and transaction_hash = <TRANSACTION_HASH>;
```

## Example 6: Fetch trace calls for a given transaction

Traces keep track of the actions that modify the internal state of the EVM.

```
select
    *
from tempo_andantino.raw.traces
where 1=1
    and transaction_hash = <TRANSACTION_HASH>;
```

## Example 7: Fetch contract details

```
select 
    *
from tempo_andantino.raw.contracts;
```