# Franklin eDNA ledger

The tamper-evident record of the Franklin national eDNA biodiversity campaign: sampling
sites, the sampling request, kits, and the chain of custody of every sample.

`ledger.jsonl` is append-only. Each line carries the SHA-256 of the line before it, so
altering any past record breaks the chain. A bare chain cannot protect its own *last*
record, so the file is periodically timestamped with [OpenTimestamps](https://opentimestamps.org),
which anchors a hash of it into the Bitcoin blockchain. Those proofs are in `anchors/`.

## Verify it yourself

Install the client once:

```bash
pip install opentimestamps-client
```

Pick any row of `anchors/log.csv`. It gives a record count, a SHA-256, and a receipt.
Because the ledger only ever grows, the state it attests is just the first N lines of
the file today:

```bash
head -n <record_count> ledger.jsonl | shasum -a 256    # must equal the sha256 column
ots info   anchors/<receipt>                            # shows the same hash
ots verify anchors/<receipt>                            # confirms the Bitcoin attestation
```

If the hashes match, every record up to that point is byte-for-byte what existed when
the timestamp was made. If they differ, the file has been altered.

`ots verify` reports the Bitcoin block and its time. A freshly made receipt reads
"pending" until the attestation is mined, which takes a few hours.

## What is not here

Personal data. The ledger identifies people only by opaque codes such as `PSN-001`;
the roster mapping those to real individuals is held privately and is not published.
