# Dumping Redis Hashes

I needed to extract all the hashes with keys that had a particular pre-fix. I started off by connecting to the redis database using [RedisInsight](https://redis.com/redis-enterprise/redis-insight/) to explore the data.

After figuring out what I needed I wrote a script to extract all of the keys with the pre-fix.

```
import IORedis from "ioredis";
import fs from "fs";

const redis = new IORedis({
  port: {{redis port}},
  host: {{redis URL}},
  password: {{redis password}},
});

const stream = redis.scanStream({
  // only returns keys following the pattern of `user:*`
  match: "user:*",
  count: 100,
});
stream.on("data", (resultKeys) => {
  for (let i = 0; i < resultKeys.length; i++) {
    console.log(resultKeys[i]);
    // save all resultKeys as rows in a CSV file
    fs.appendFileSync("./user-dump.csv", resultKeys[i] + "\n");
  }
});
stream.on("end", () => {
  console.log("all keys have been visited");
});
```

Then I extracted the data from each key. Because the data I needed is in an array I used `HGETALL`. ioredis has a nice streaming function that converts the response into an object for easy access to the key/value pairs:

```
...
import { parse } from "csv-parse";

let count = 0;

fs.createReadStream("./user-dump.csv")
  .pipe(parse({ delimiter: "\n" }))
  .on("data", (row) => {
    console.log(count);
    count++;
    console.log(row[0]);
    redis.hgetall(row[0]).then((result) => {
      if ("username" in result) {
        let row = `${result.email},${result.username},${result.name}\n`;
        fs.appendFileSync("./usernames.csv", row);
      }
    });
  })
  .on("end", function () {
    console.log("finished");
    console.log(`${count} rows`);
  })
  .on("error", function (error) {
    console.log(error.message);
  });
```
