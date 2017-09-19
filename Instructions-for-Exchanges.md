# Instructions for Exchanges

We here describe how to interface your exchange with the YOYOW blockchain step-by-step.

## Preparation

### Hardware

The recommended hardware by now is a VPS with 2GB of RAM and 20GB HDD. Single core is OK.

Supported platforms:
* Ubuntu 16.04 LTS 64 bit
* Windows Servers 64 bit

We'll take Ubuntu as an example in this document.

### Services

Disable the default time-syncing daemon (timedated) and install NTPD:
```
sudo timedatectl set-ntp false
sudo apt-get -y install ntp
```

### Account

Register an account with https://wallet.yoyow.org. Instruction is [here](https://steemit.com/cn/@peterchen145/yoyow-online-wallet-sign-up-tutorial). Save your account ID number for later use.

Get the private keys of your account. We'll need 3 keys for integration:
* Active key: for transferring funds out
* Secondary key: for collecting points to pay fees
* Memo key: for encrypting/decrypting memos

### Business Logic

An exchange uses an account (or more) for processing deposits/withdrawals.
* Deposits: assign a unique identifier to every customer, for example salted hash of user ID. Every customer transfer funds to the exchange's account with different identifiers as `memo`, so the exchange can know a deposit is for which customer.
* Withdrawals: transfer funds from the exchange's account to an account the customer requested, with an optional memo that the customer may have requested. An optional memo is needed, because some customers may want to withdraw funds to another exchange directly.

## Installation

As of writing, only binaries are available for download ([link](https://github.com/yoyow-org/yoyow-core/releases)). After downloaded, extract the files.

```
wget https://github.com/yoyow-org/yoyow-core/releases/download/v0.1.0-170906/yoyow20170906.tgz
tar xzf yoyow20170906.tgz
mv yoyow20170906 yoyow
wget https://github.com/yoyow-org/yoyow-core/releases/download/v0.1.0a-170908/yoyow_client_20170908.gz
gunzip -k yoyow_client_20170908.gz
mv yoyow_client_20170908 yoyow/yoyow_client
cd yoyow
```

## Start YOYOW Node

The node need to be always running. One way to archive this is to run it with `screen`. If you don't have `screen`, install it:
```
sudo apt-get -y install screen
```

Start the node inside `screen`:
```
screen -S yoyow_node
./yoyow_node --rpc-endpoint 127.0.0.1:8090 --replay-blockchain
```

Note:
1. Start the node with `--rpc-endpoint` so we can interact with it via RPC call. In the example the node will listen on address `127.0.0.1` and port `8090`.
2. There is a `--track-account` option ported from BitShares, which is useful for exchanges, but it's now broken in YOYOW. It will be fixed in the future. With this option, the node will use less RAM. Anyway, in the beginning, the node won't use much RAM even without that option.
3. There is a bug which causes syncing issues if the node is not started with `--replay-blockchain` option, so we need to start with that option as a temporary workaround. We're looking into the issue. After it's fixed, no need to start with this option every time.

The node then will download blocks from the p2p network. When it's done (in sync), in the console there will be new messages showing every 3 seconds, like these:
```
789216ms th_a       application.cpp:574           handle_block         ] Got block: #355797 00056dd54878c05849e2dcd731c9ee364398b0d2 time: 2017-09-18T19:13:09 latency: 216 ms from: 27662/init8  irreversible: 355787 (-10)
792527ms th_a       application.cpp:574           handle_block         ] Got block: #355798 00056dd613f39f1ad6c0c3fdb00a56c60f44ab1c time: 2017-09-18T19:13:12 latency: 526 ms from: 499381505/yoyo499381505  irreversible: 355788 (-10)
```

## Start YOYOW Client

### First Run
Start another `screen` session, run `yoyow_client` inside:
```
screen -S yoyow_client
./yoyow_client -s ws://127.0.0.1:8090/ -H 127.0.0.1:8091
```

Note:
1. Use `-s` option to connect the client to the node.
2. Use `-H` option to start a HTTP-RPC service so we can interact with the client from another process, E.G. a script to process funds deposit/withdrawal.
3. The node won't start listening on the RPC port until finished replaying, so please be patient in this case.
4. You can connect multiple clients to one node, but don't use the same `-H` option.

For the first time when running `yoyow_client`, after connected to the node, it will show:
```
Please use the set_password method to initialize a new wallet before continuing
new >>>
```

So we set a password, it will be used to create and encrypt a wallet file:
```
new >>> set_password my-password
set_password my-password
null
locked >>>
```

When there is already a wallet file, if we run `yoyow_client`, it will show `locked >>>` as well.

Then we unlock the wallet:
```
locked >>> unlock my-password
unlock my-password
null
unlocked >>>
```

Now we import the 3 private keys into the client, they will be encrypted then saved in the wallet file. The syntax is:
```
import_key [account_ID] [wif_private_key]
```
Assume your account ID is `123456789`, and you have the keys, we need to execute `import_key` command 3 times, one time for one key. For example:
```
unlocked >>> import_key 123456789 5Hqwx3xXMYZ55Pko9nzw34234234nXHcGfNQjNEL23424w7Py
import_key 123456789 5Hqwx3xXMYZ55Pko9nzw34234234nXHcGfNQjNEL23424w7Py
2993104ms th_a       wallet.cpp:820                save_wallet_file     ] saving wallet to file wallet.json
true
unlocked >>> import_key 123456789 5JKoYzQ4sYZoDYwreyrsfsd32466MsCFNoxRE23nExaRi6SY3
import_key 123456789 5JKoYzQ4sYZoDYwreyrsfsd32466MsCFNoxRE23nExaRi6SY3
2993104ms th_a       wallet.cpp:820                save_wallet_file     ] saving wallet to file wallet.json
true
unlocked >>> import_key 123456789 5HttjgBSb45368989etfhsserVtt69cWcExteq6RktpAYXNTT
import_key 123456789 5HttjgBSb45368989etfhsserVtt69cWcExteq6RktpAYXNTT
2993104ms th_a       wallet.cpp:820                save_wallet_file     ] saving wallet to file wallet.json
true
unlocked >>>
```

### The `info` Command

We can check network status with `info` command:
```
unlocked >>> info
info
{
  "head_block_num": 377867,
  "head_block_id": "0005c40b41f6d79d762b1ff81c7affc7ae82a894",
  "head_block_time": "2017-09-18T19:37:39",
  "head_block_age": "0 second old",
  "last_irreversible_block_num": 377857,
  "chain_id": "3505e367fe6cde243f2a1c39bd8e58557e23271dd6cbf4b29a8dc8c44c9af8fe",
  "participation": "100.00000000000000000",
  "active_witnesses": [[
  ...
}
```

### The `get_block` Command

We can get the details of a specified block with `get_block` command. The syntax is:
```
get_block [block_number]
```
For example:
```
unlocked >>> get_block 1
```

### The `get_full_account` Command

We can check the account info with `get_full_account` command:
```
unlocked >>> get_full_account 123456789
get_full_account 123456789
{
  "account": {
    "uid": 123456789,
    ...
  },
  "statistics": {
    "owner": 123456789,
    "total_ops": 30220,
    "prepaid": 0,
    "csaf": 37424828,
    "core_balance": "44672014515",
    "core_leased_in": 0,
    ...
```

Note: the "statistics" data is useful for integration.
* "csaf" is points, which will be used to pay transaction fees. Balances have 5 decimal digits in YOYOW, and the currency is `YOYO`, so `"csaf": 37424828` means `374.24828 YOYO`.
* "core_balance" is balance of the account. `"core_balance": "44672014515"` means `446,720.14515 YOYO`.
* Please be aware numbers will be surrounded with quotation marks when bigger than `2^32`, as shown above for `core_balance` but not for `csaf`.

### The `transfer` Command

We can use the `transfer` command to transfer funds. The syntax is:
```
transfer [from] [to] [amount] YOYO [broadcast]
```
For example:
```
unlocked >>> transfer 123456789 987654321 1.2345 YOYO true
```

Note:
* `amount` can only have at most 5 decimal digits.
* if set `broadcast` to `true`, the signed transaction will be broadcast to the p2p network. Use `false` for testing.

### The `get_transaction_id` Command

We can use the `get_transaction_id` command to get the hash of a transaction. The syntax is:
```
get_transaction_id [transaction_in_json]
```
This command is useful for integration.

### The `get_relative_account_history` Command

We can use the `get_relative_account_history` command to check our transaction history. The syntax is:
```
get_relative_account_history [account] [operation_type] [start] [limit] [end]
```
For example:
```
unlocked >>> get_relative_account_history 123456789 null 1 10 10
unlocked >>> get_relative_account_history 123456789 0 11 10 20
```

Note:
* For `operation_type`, use `null` to get all operations, use `0` to get transfers only.
* For `end`, use `0` to get the most recent records.
* Result will be in range of `[start, end]`; if `limit` is smaller than the number of records in `[start, end]`, the latest records will be returned.
* Result is sorted in "latest first" order.

### The `collect_csaf` Command

We can use the `collect_csaf` command to collect points which will be needed to pay transaction fees. The syntax is:
```
collect_csaf [from_account] [to_account] [amount] YOYO [broadcast]
```
For example:
```
unlocked >>> collect_csaf 123456789 123456789 10 YOYO true
```

Note:
* If you have some YOYO in your account, it will accumulate points as time goes by. The accumulation speed has a linear relationship with account balance. Usually for exchanges the accumulated points should be enough to pay transaction fees.
* Points need to be collected before can be used to pay transaction fees, so we have this command.
* Although funds in balance can be used to pay transaction fees as well in the back end, current implementation of `yoyow_client` will only try to pay fees with points (except raw-transaction signing). If you don't have enough points in the account, most commands will fail. So it's important to keep a certain amount of points in the account.

## Access the Client via HTTP-RPC

When HTTP-RPC is enabled, we can access the client via HTTP-RPC call. All commands are usable. For example:
```
curl -d '{"jsonrpc": "2.0", "method": "info", "params": [], "id": 1}' http://127.0.0.1:8091/rpc
curl -d '{"jsonrpc": "2.0", "method": "transfer", "params": [123456789,123456789,1,"YOYO",null,true], "id": 1}' http://127.0.0.1:8091/rpc
curl -d '{"jsonrpc": "2.0", "method": "get_relative_account_history", "params": [123456789,0,1,10,10], "id": 1}' http://127.0.0.1:8091/rpc
```

Note:
* Use `http` as protocol
* Request `/rpc` but not `/`
* Not like the interactive CLI, the results of HTTP-RPC calls are formatted in json
* In the request, amounts have 5 decimal digits; in the response, amounts have no decimal point, instead, amounts are multiplied by `10^5`.

## Process Deposits

### Check Node Status

Get the `last_irreversible_block_num` data with the `info` command (via HTTP-RPC). Only blocks earlier than this block is reliable.

### Check Account History

1. Firstly, Use `get_relative_account_history` command/API to get the latest sequence number:
```
curl -d '{"jsonrpc": "2.0", "method": "get_relative_account_history", "params": [123456789,0,0,1,0], "id": 1}' http://127.0.0.1:8091/rpc
```

If the result, `response["result"]` is empty, means we have no deposit at all. If it's not empty, we get `response["result"][0]["sequence"]` as `maximum_sequence`.

If `maximum_sequence` is bigger than the last sequence we've saved, it means there are new records to be processed.

2. Use `get_relative_account_history` command/API to check for new records. For example, if last recorded `sequence` is `100`, and `maximum_sequence` is `200`, we can check from `101`, for at most `100` records, to `200`:
```
curl -d '{"jsonrpc": "2.0", "method": "get_relative_account_history", "params": [123456789,0,101,100,200], "id": 1}' http://127.0.0.1:8091/rpc
```

The returned result, `result=response["result"]`, is an array. If the array is empty, it means no new deposit. If it's not empty, the N'th record `result[N]` should be like:
```
    {
      "memo": "a1b2c3d4",
      "description": "Transfer 100 YOYO from 204501630 to 123456789 -- Memo: a1b2c3d4   (Fee: 0.20898 YOYO)",
      "sequence": 101,
      "op": {
        "op": [
          0,
          {
            "fee": {
              "total": {
                "amount": 20898,
                "asset_id": 0
              },
              "options": {
                "from_balance": {
                  "amount": 20898,
                  "asset_id": 0
                }
              }
            },
            "from": 204501630,
            "to": 123456789,
            "amount": {
              "amount": 10000000,
              "asset_id": 0
            },
            "memo": {
              "from": "YYW6U528P71X6V87765245356aPBPpDwwRp7urUiXYtFLHmrXRsN3u",
              "to": "YYW5eA89yqwerhdfghrjtr3452376trtyU6LD7a1kmvwYa5h51rDxr",
              "nonce": "3457645755345345",
              "message": "0938457345937abcdef3098945"
            },
            "extensions": {
              "from_balance": {
                "amount": 10000000,
                "asset_id": 0
              },
              "to_balance": {
                "amount": 10000000,
                "asset_id": 0
              }
            }
          }
        ],
        "result": [
          0,
          {
          }
        ],
        "block_timestamp": "2017-09-08T11:14:15",
        "block_num": 223355,
        "trx_in_block": 0,
        "op_in_trx": 0,
        "virtual_op": 8795
      }
    },
```

* Get `result[N]["op"]["block_num"]`, if it's smaller than `last_irreversible_block_num`, then it's reliable, need to be processed.
* Get `result[N]["op"]["op"][0]`, if it's `0`, then it's a transfer. (Although it should always be `0` if we request transfers only)
* Get `result[N]["op"]["op"][1]["to"]`, if it's same as our account ID, then this transfer is a deposit
* Check if `result[N]["op"]["op"][1]["amount"]["asset_id"]` is `0`, means it's `YOYO` asset
* Get `result[N]["op"]["op"][1]["amount"]["amount"]`, it's the amount. Remember to add the decimal point (5 digits).
* Get `result[N]["memo"]`, it should have been decrypted already, it's an identifier to the customer, process it.
* Save `result[N]["sequence"]` as last processed sequence
* Get `result[N]["op"]["trx_in_block"]` for later use
* Use `get_block` command/API to get this transfer's transaction ID/Hash (change the parameter to the `block_num`):
```
curl -d '{"jsonrpc": "2.0", "method": "get_block", "params": [160000], "id": 1}' http://127.0.0.1:8091/rpc
```
  From the response, `new_response`, save `new_response["result"]["transaction_ids"][trx_in_block]` as the transaction ID/hash of this deposit for future use.

Note:
* To be able to decrypt memo, the client need to be `unlocked`, and the memo private key need to be in the wallet.

## Process Withdrawal Requests

### Check Node Status

To be safe, we can only process withdrawal requests when node status is normal.

Check with the `info` command/API.

* `head_block_time` should be no more than 15 seconds old
* `participation` should be more than `80`, which means 80% of block producers are online

### Check Account Balance and Points

Check with the `get_full_account` command/API.

If points are not enough for paying transaction fee, collect more points with `collect_csaf` command/API.

### Send funds

1. Use the `transfer` command/API to send out funds.

Note: take care of decimal digits.

Save the returned json for future use.

2. Get the transaction ID/hash with `get_transaction_id` command/API, save it for future use.

### Re-check / Follow-up

Similar to the deposit processing steps, when found an outgoing transfer, save the transaction ID/hash, block number and etc for future use.

### Trouble shooting

Every transaction has an `expiration` field. If the transaction hasn't been included in any block for some reason, and the timestamp of block whose number is `last_irreversible_block_num` is later than the `expiration` field of the transaction, the transaction won't be included in current chain, so it's safe to try to transfer again.

## Sample Code (Ruby)
```
require 'time'
require 'bigdecimal'
require "httpclient"
require "json"
require 'mysql2'
require 'rufus-scheduler'

$my_yoyow_check_config = {
  "head_age_threshold" => 15,              #seconds
  "participation_rate_threshold" => 79.999 #percent
}
$my_exchange_config = {
  "yoyow_account" => "123456789", 
  "yoyow_asset_id" => 0,
  "mysql_config" => {
    "host" => "192.168.1.1",
    "user" => "exchange",
    "pass" => "Yoyow12345",
    "db"   => "exchange"
  }
}
def my_yoyow_config
  {
    "uri"  => 'http://127.0.0.1:8091/rpc',
    "user" => "",
    "pass" => ""
  }
end

def yoyow_post (data:{}, timeout:55)
  if data.nil? or data.empty?
    return
  end

  client = HTTPClient.new
  client.connect_timeout=timeout
  client.receive_timeout=timeout

  myconfig = my_yoyow_config
  uri  = myconfig["uri"]
  user = myconfig["user"]
  pass = myconfig["pass"]

  client.set_auth uri, user, pass

  begin
    response = client.post uri, data.to_json, nil
    response_content = response.content
    response_json = JSON.parse response_content
    return response_json
  rescue Exception => e
    print "yoyow_post error: "
    puts e
  end
end

# params is an array
def yoyow_command (command:nil, params:nil, timeout:55)
  if command.nil? or params.nil?
    return
  end
  data = {
    "jsonrpc" => "2.0",
    "method"  => command,
    "params"  => params,
    "id"      => 0
  }
  return yoyow_post data:data, timeout:timeout
end

#main
if __FILE__ == $0

  logger = Logger.new(STDOUT)
  gwcf = $my_exchange_config
  sqlcf = gwcf["mysql_config"]
  sqlc = Mysql2::Client.new(:host=>sqlcf["host"],:username=>sqlcf["user"],:password=>sqlcf["pass"],:database=>sqlcf["db"])
  yoyow_account = gwcf["yoyow_account"]
  yoyow_asset_id = gwcf["yoyow_asset_id"]

  # run
  scheduler = Rufus::Scheduler.new
  scheduler.every '10', first: :now, overlap: false do
   begin
    # [YOYOW] check if current node or current network status is ok
    r = yoyow_command command:"is_locked", params:[]
    if r["result"] == true
      msg = "[YOYOW] Warning: wallet is locked."
      logger.warn msg
      next
    end

    r = yoyow_command command:"info", params:[]
    yoyow_head_num = r["result"]["head_block_num"]
    yoyow_head_time = r["result"]["head_block_time"]
    yoyow_head_age_sec = (Time.now.utc - Time.parse(yoyow_head_time + 'Z')).to_i
    yoyow_lib = r["result"]["last_irreversible_block_num"]
    yoyow_participation_rate = BigDecimal.new(r["result"]["participation"])

    logger.info "[YOYOW] head block #%d, LIB #%d, time %s, age %d s, NPR %.2f %" %
                 [ yoyow_head_num, yoyow_lib, yoyow_head_time, yoyow_head_age_sec, yoyow_participation_rate.to_f ]
    if yoyow_head_age_sec >= $my_yoyow_check_config["head_age_threshold"]
      msg = "[YOYOW] Warning: head block age %d seconds." % yoyow_head_age_sec
      logger.warn msg
      next
    end
    if yoyow_participation_rate <= $my_yoyow_check_config["participation_rate_threshold"]
      msg = "[YOYOW] Warning: participation rate %.2f %" % yoyow_participation_rate.to_f
      logger.warn msg
      next
    end

    # [SQL] reconnect
    if sqlc.nil? or sqlc.closed?
      sqlc = Mysql2::Client.new(:host=>sqlcf["host"],:username=>sqlcf["user"],:password=>sqlcf["pass"],:database=>sqlcf["db"])
    end

    # [YOYOW] get last processed number
    yoyow_next_seq = 1
    sql = "SELECT next_seq_no FROM yoyow_info WHERE monitor_account = " + yoyow_account
    sqlr = sqlc.query(sql)
    if sqlr.size == 0
      sqlc.query("insert into yoyow_info(monitor_account,next_seq_no) values(" + yoyow_account + ",1)")
    else
      sqlr.each do |row|
        # do something with row, it's ready to rock
        yoyow_next_seq = row["next_seq_no"]
      end
    end
    logger.info "[YOYOW] next seq to process = %d" % yoyow_next_seq

    # [YOYOW] process new history
    sql = "insert into yoyow_his(monitor_account,seq_no,from_account,to_account,amount,asset_id," +
                              "decrypted_memo,description,block_num,block_time,trx_in_block,op_in_trx,virtual_op,trx_id," +
                              "process_status) " +
                              "values(?,?,?,?,?,?,?,?,?,?,?,?,?,?,?)"
    stmt = sqlc.prepare(sql)
    yoyow_last_block_num = 0
    yoyow_last_block = nil
    yoyow_max_seq = 0
    r = yoyow_command command:"get_relative_account_history", params:[yoyow_account,0,0,1,0]
    if not r["result"].empty?
      yoyow_max_seq = r["result"][0]["sequence"]
    end
    r = yoyow_command command:"get_relative_account_history", params:[yoyow_account,0,yoyow_next_seq,10,yoyow_next_seq+9]
    catch :reached_lib do
     while yoyow_next_seq <= yoyow_max_seq do
      r["result"].reverse.each{ |rec|
         seq_no = rec["sequence"]
         block_num = rec["op"]["block_num"]
         throw :reached_lib if block_num >= yoyow_lib # don't process records later than LIB
         yoyow_next_seq = seq_no + 1
         next if rec["op"]["op"][0] != 0 # not transfer
         logger.info "[YOYOW]" + rec.to_s
         sqlr = sqlc.query("select 1 from yoyow_his where monitor_account = " + yoyow_account + " and seq_no = " + seq_no.to_s)
         if sqlr.size > 0
           logger.info "[YOYOW] skipping because [account,seq] = [%s,%d] is already in db" % [ yoyow_account, seq_no ]
           next
         end
         if block_num != yoyow_last_block_num
           yoyow_last_block_num = block_num
           r1 = yoyow_command command:"get_block", params:[block_num]
           yoyow_last_block = r1["result"]
         end
         from_account = rec["op"]["op"][1]["from"].to_s
         to_account = rec["op"]["op"][1]["to"].to_s
         amount = rec["op"]["op"][1]["amount"]["amount"].to_i
         asset_id = rec["op"]["op"][1]["amount"]["asset_id"]
         decrypted_memo = rec["memo"]
         description = rec["description"]
         block_time = rec["op"]["block_timestamp"]
         trx_in_block = rec["op"]["trx_in_block"]
         op_in_trx = rec["op"]["op_in_trx"]
         virtual_op = rec["op"]["virtual_op"]
         trx_id = yoyow_last_block["transaction_ids"][trx_in_block]
         process_status = 0 # new
         if from_account == yoyow_account
           process_status = 2 # out
           sqlc.query("update withdraw_his set process_status = 23, out_block_num = " + block_num.to_s +
                             " where out_platform = 'yoyow' " +
                             "and out_trx_id = '" + trx_id + "' and process_status = 22")
         elsif asset_id != yoyow_asset_id
           process_status = 101 # bad asset
         elsif decrypted_memo.nil? or decrypted_memo.length == 0
           process_status = 102 # empty memo
         else
           # parse memo
           memo_to_parse = decrypted_memo
           # check if the memo is valid
           if memo_to_parse.ok?
               process_status = 11 # good memo
               # to process
           else
               process_status = 104 # bad memo
           end
         end
         stmt.execute(yoyow_account,seq_no,from_account.to_i,to_account.to_i,amount,asset_id,
                      decrypted_memo,description,block_num,block_time,trx_in_block,op_in_trx,virtual_op,trx_id,
                      process_status)
      }
      r = yoyow_command command:"get_relative_account_history", params:[yoyow_account,0,yoyow_next_seq,10,yoyow_next_seq+9]
     end # while
    end #catch
    sqlc.query("update yoyow_info set next_seq_no = " + yoyow_next_seq.to_s + " where monitor_account = " + yoyow_account)

    # [SQL] process new withdrawals
    sql = "select * from withdraw_his where process_status = 11 order by seq_no"
    sqlr = sqlc.query(sql)
    if sqlr.size > 0
     catch :collected_csaf do
      # [YOYOW] check account statistics
      r = yoyow_command command:"get_full_account", params:[yoyow_account]
      stats = r["result"]["statistics"]
      if stats["csaf"] < 2000000 # 20 YOYO
        logger.info "[YOYOW] collecting CSAF"
        r1 = yoyow_command command:"collect_csaf", params:[yoyow_account,yoyow_account,20,"YOYO",true]
        logger.info r1.to_s
        if not r1["error"].nil?
          msg = "[YOYOW] Warning: insufficient CSAF"
          logger.warn msg
        end
        throw :collected_csaf # do nothing until next loop
      end
      # [YOYOW] process
      avail_balance = BigDecimal.new(stats["core_balance"]) -
                      BigDecimal.new(stats["total_witness_pledge"]) -
                      BigDecimal.new(stats["total_committee_member_pledge"])
      sqlr.each do |row|
        out_amount = row["out_amount"] # raw data, should / 100000
        if avail_balance >= out_amount
          row_id = row["row_id"]
          out_addr = row["out_address"]
          out_memo = row["out_memo"]
          float_out_amount_str = "%d.%05d" % [ out_amount / 100000, out_amount % 100000 ]
          logger.info "[SQL] ready to process:  -> YOYOW(%s),  %s YOYO" %
                      [ out_addr,  float_out_amount_str ]
          sqlc.query("update withdraw_his set process_status = 21 where row_id = " + row_id.to_s)
          logger.info "[YOYOW] sending..."
          out_time = Time.now.utc.to_s
          r1 = yoyow_command command:"transfer", params:[yoyow_account,out_addr,float_out_amount_str,"YOYO",out_memo,true]
          logger.info r1.to_s
          if not r1["error"].nil? # fail
            logger.warn "[YOYOW] error while sending"
            sql1 = "update withdraw_his set process_status = 201, out_time = ?, out_detail = ? where row_id = ?"
            stmt1 = sqlc.prepare(sql1)
            stmt1.execute(out_time, r1["error"].to_s, row_id)
          elsif not r1["result"].nil? # ok
            logger.info "[YOYOW] done sending"
            trx = r1["result"]
            r2 = yoyow_command command:"get_transaction_id", params:[trx]
            trx_id = r2["result"]
            sql1 = "update withdraw_his set process_status = 22, out_time = ?, out_trx_id = ?, out_detail = ? where row_id = ?"
            stmt1 = sqlc.prepare(sql1)
            stmt1.execute(out_time, trx_id, trx.to_s, row_id)
          else # should not be here, just to be safe
            sql1 = "update withdraw_his set process_status = 202, out_time = ?, out_detail = ? where row_id = ?"
            stmt1 = sqlc.prepare(sql1)
            stmt1.execute(out_time, r1.to_s, row_id)
          end
          break # process one item every (outer) loop
        else
          msg = "[YOYOW] Warning: insufficient balance"
          logger.warn msg
        end # if avail_balance
      end # sqlr.each
     end # catch CSAF
    end # if sqlr.size > 0
   rescue Exception => e
     logger.error e.to_s
     logger.error e.backtrace
     msg = "Exception: %s" % e.to_s
   end
  end

  scheduler.join

end
__END__

```