# clj-kinesis-worker

Thin wrapper around the Amazon Kinesis Client Library, more specifically the Worker class.

Developing workers using the KCL is preferrable to using the API directly, because
the library takes care of things like distributing shards among workers and providing
a distributed failover mechanism. See the [docs] [1] for more information.
[This talk] [2] is also pretty informative!

## Usage

You will need to implement a protocol:

```clojure
;; (defprotocol RecordProcessor
;;   (initialize [this shard-id])
;;   (process-records [this shard-id records checkpointer])
;;;  (shutdown [this shard-id checkpointer reason]))


(defrecord TestProcessor []
  RecordProcessor
  (initialize [_ shard-id] ...)
  (process-records [_ shard-id records checkpointer] ...)
  (shutdown [_ shard-id checkpointer reason] ...)

(defn new-processor [] (->TestProcessor))
```

A worker can then be created:

```clojure
(def worker
  (clj-kinesis-worker.core/create-worker
    {:region               "eu-west-1"
     :stream-name          "some-stream"
     :app-name             "some-app"
     :processor-factory-fn new-processor}))
```

## Development

KCL uses both DynamoDB and Kinesis to keep track of and process events. Both DynamoDB and Kinesis can be mocked locally:

```
$> npm install -g dynalite
$> npm install -g kinesalite
$> dynalite --port 4567 &
$> kinesalite --port 4568 &
```

The corresponding worker configuration:

```clojure
(clj-kinesis-worker.core/create-worker
  {:kinesis              {:endpoint "http://localhost:4568"}
   :dynamodb             {:endpoint "http://localhost:4567"}
   :region               "eu-west-1"
   :stream-name          "some-stream"
   :app-name             "some-app"
   :processor-factory-fn new-processor}))

```

You can now use another library like [clj-kinesis-client] [3] to feed the pipeline from one end.

Note that KCL still has a CloudWatch dependency. So access to AWS needs to be secured,
for example using environment variables like `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.
The `create-worker` function will use the default AWS credentials provider chain for
authentication.

## TODO

Client configuration like max retries, timeouts.

## License

Copyright © 2016 Johannes Staffans

Distributed under the Eclipse Public License either version 1.0 or (at
your option) any later version.

[1]: http://docs.aws.amazon.com/kinesis/latest/dev/developing-consumers-with-kcl.html
[2]: https://www.youtube.com/watch?v=AXAaCG2QUkE
[3]: https://github.com/adtile/clj-kinesis-client