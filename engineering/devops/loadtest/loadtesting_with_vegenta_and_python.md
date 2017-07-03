
[Source](https://serialized.net/2017/06/load-testing-with-vegeta-and-python/ "Permalink to HTTP Load Testing with Vegeta (and a dash of Python)")

# HTTP Load Testing with Vegeta (and a dash of Python)

When trying to make scalable computer systems, it's almost impossible to fully simulate all the ways things can break. However, it's very easy to simulate _some_ of sorts of things that may break you – and it's well worth learning at least the easy lessons early and often. (Especially if it prevents a late night freaked out phone call 😴).

I speak to a lot of people who want to have a scalable website or service – it's the first thing they mention in a discussion – but pre-production load testing is a suprisingly rare practice.

Just as you don't really know if you're doing backups until you test a restore – you don't really know if your website will handle a lot of traffic unless you send a lot of traffic at it!

These days there are a number of commercial providers who offer load testing services, and many of these are very good. But there are also many powerful open source tools that are often sufficient, and even work in places SaaS producs don't, like testing things that live off the public internet.

I've used and enjoyed many such tools (apachebench, siege, gatling, wrk, goad, boom, locust.io), and they have different strengths. But my favorite recently is [vegeta][1].

Update: I'm told [artillery][2] deserves a look as well, and supports the load ramping feature natively.

The key vegeta features I enjoy are:

* A simple and composable command line interface
* A config file format which trivially allows setting custom headers, which is key to working with pre-production services, and allows testing multiple endpoints simultaneously
* The ability to run multiple tests in parallel across different hosts, increasing the total test volume possible
* It generates a constant request rate, a feature found in few other tools, but also a key to finding true operational limitations.
* Because it's Go, it's compiled to a single binary, making it trivial to drop into machines if you're doing scale out or remote node testing.
* It also has an excellent library interface, allowing it to be a component of custom tooling.

This post talks about using vegeta AND python. In many cases, just using vegeta alone is totally sufficient. However, I found that using python to drive vegeta and visualize the output has been very helpful.

### Load testing with Vegeta

For this example, I'm testing a [basic Wordpress install][3] started with Docker Compose, and configured with a theme and some starter content.

The target file syntax is straightforward, but very flexible. Here, I'm testing a few different endpoints in the site. Vegeta will round robin between them.

`targets.txt`:
    
    
    GET http://localhost:8755/
    GET http://localhost:8755/2017/04/27/hello-world/
    GET http://localhost:8755/category/uncategorized/
    

A quick note on the syntax here: If, for example, you wanted to test something that was behind an Elastic Load Balancer, but didn't yet have DNS pointed at it (like in a Blue/Green deployment strategy), you can easily do this with Vegeta.
    
    
    GET http://my-load-balancer-1234567890.us-west-2.elb.amazonaws.com/mypage
    Host: myrealdomain.com
    

This is much nicer than monkeying with `/etc/hosts`, and you can also use this to test multiple virtual hosts that might be behind the same load balancer.

One feature vegeta doesn't have (at least for now, there's been an issue open requesting it for a few years) is to ramp up request rate over time, to find out where/how a system will break. However, the command line interface is so well designed, it's actually quite easy to do with basic unix techniques.
    
    
    $ for rate in $(seq 1 3 80); do
        vegeta attack -duration=90s -rate $rate -targets targets.txt |
        vegeta dump -dumper csv |
        sed "s/$/,$rate/" >> loadtest.csv
    done
    

This will go through various rates from 1 to 80, adding 3 requests per second every 90 seconds. It will use the above `targets.txt`, and pipe the results to `vegeta dump`, which will give us the raw information back in CSV format. Since it's just a CSV, it's easy to add an additional field at the end of each line with `sed` for the current rate of the test. The results from each test are appended to the same CSV file.

However, that CSV file is not incredibly readable, especially since humans don't often think in nanosecond timestamps.

### Using Python to analyze the results.

So, here's where Python comes in handy, especially (at least when prototyping) with the [Jupyter Notebook][4].
    
    
    # Read in the custom CSV made by vegeta with the extra 'request rate' column
    csv = pd.read_csv('loadtest.csv', header=None, names=["timestamp", "code", "latency", "bytesout", "bytesin", "error", "rate"], index_col=[0])
    
    # Pandas nicely knows how to handle nanosecond epoch timestamps
    csv.index = pd.to_datetime(csv.index, unit='ns')
    
    # Convert latency nanoseconds -> seconds for legibility
    csv['latency'] = csv.latency.apply(lambda x: x / 1e9)
    
    # resample data set to 30 second 'chunks' with key columns
    # Use the 90th percentile to help remove outliers
    df = pd.DataFrame()
    df['latency'] = csv.latency.resample('30S').agg(lambda x: x.quantile(.90))
    df['rate'] = csv.rate.resample('30S').max()
    
    # plot it, with the request rate on it's own axis
    df.rate.plot()
    df.latency.plot(secondary_y=True)
    

8 lines of code gives a custom analysis and visualization of the latency as a function of request rate. The blue line is the requests/second, and the green line is the latency.

![][5]

Mission accomplished, we found the limitations of this configuration. 💥 (The reason there is a hard cap at 30 second response time is because vegeta's default timeout is 30 seconds.)

It's easy to find exactly where it fails:
    
    
    df.groupby('rate')['latency'].mean()
    

This prints the means of each latency. Normally I wouldn't use mean for latency, but since the 90th percentile has already been applied, it'll play. Zooming into the interesting part of the output:
    
    
    r/sec   latency (s)
    58      0.082885
    61      0.163670
    64      0.194260
    67      1.344821
    70      2.568766
    73      3.139010
    76     19.530751
    

At 58 requests/second, things are fine, dare I say snappy. As the request/second count increases beyond that, things degrade quickly. This, at a glance, is one of the characteristics that makes system operations so challenging. Many of these systems behave perfectly in a very wide band of utilization, but the gap between "running great" and "total failure" can be relatively tiny.

### Using Vegeta's API for process automation

Having the test data in a Pandas dataframe is a useful tool to understand the general performance of the system and for getting custom visualizations. What if we wanted to introduce an automated scalability check as part of a CI/CD process? It would be possible to do it with this approach, but once it's become clear "we are looking for the point at which the system starts to fall over", the graphs and other data become less useful. It would be great to be able to say "don't deploy code automatically when it breaks at a lower number of requests/second than the previous release."

The previous approach gave a very comprehensive picture of how the system behaved at various utilization rates, but it was a pretty ineffient way to find the answer.

Vegeta's library API to the rescue. This can be solved simply with a small custom go program:

Full source: [vegeta_breaker.go][6]

At it's heart, it's a Computer Science 101 binary search:
    
    
    // first, find the point at which the system breaks
    for {
        if testRate(rate, sla) {
            okRate = rate
            rate *= 2
        } else {
            nokRate = rate
            break
        }
    }
    
    // next, do a binary search between okRate and nokRate
    for (nokRate - okRate) > 1 {
        rate = (nokRate + okRate) / 2
        if testRate(rate, sla) {
            okRate = rate
        } else {
            nokRate = rate
        }
    }
    

The vegeta API makes implementing the `testRate` function, which enables this simplicity, a breeze.
    
    
    func testRate(rate int, sla time.Duration) bool {
    	duration := 15 * time.Second
    	targeter := vegeta.NewStaticTargeter(vegeta.Target{
    		Method: "GET",
    		URL:    "http://localhost:8755/",
    	})
    	attacker := vegeta.NewAttacker()
    	var metrics vegeta.Metrics
    	for res := range attacker.Attack(targeter, uint64(rate), duration) {
    		metrics.Add(res)
    	}
    	metrics.Close()
    	latency := metrics.Latencies.P95
    	if latency > sla {
    		fmt.Printf("💥  Failed at %d req/sec (latency %s)n", rate, latency)
    		return false
    	}
    	fmt.Printf("✨  Success at %d req/sec (latency %s)n", rate, latency)
    	return true
    }
    

And that is it! Now it's easy to run:
    
    
    $ go run vegeta_breaker.go
    ✨  Success at 20 req/sec (latency 73.073787ms)
    ✨  Success at 40 req/sec (latency 48.128922ms)
    💥  Failed at 80 req/sec (latency 12.031913894s)
    ✨  Success at 60 req/sec (latency 100.134386ms)
    💥  Failed at 70 req/sec (latency 1.313739719s)
    ✨  Success at 65 req/sec (latency 116.262857ms)
    ✨  Success at 67 req/sec (latency 237.865743ms)
    ✨  Success at 68 req/sec (latency 515.387628ms)
    ✨  Success at 69 req/sec (latency 517.864623ms)
    ➡️  Maximum Working Rate: 69 req/sec
    

And in 2.5 minutes (instead of the 40 before) this actually gave a more precise result for exactly where the system would fail. To integrate this into a deployment process would be very simple, as the build would produce a static binary – no need to even have vegeta itself installed, as this tool would include it.

Note, everything's hard coded here, and only latency is checked.

For real use, you'd probably want to:

* make endpoints & thresholds more configurable than hard coded
* Check for HTTP error codes, not just latency.
* Give the system a 'cooldown' window after driving it to failure
* Produce an output that's more meaningful to your system, e.g. outputting JUnit for Jenkins to consume

Vegeta. It truly is, as the website says, [over 9000][7].

[1]: https://github.com/tsenart/vegeta
[2]: https://artillery.io/
[3]: https://docs.docker.com/compose/wordpress/
[4]: http://jupyter.org/
[5]: https://serialized.net/images/vegeta/overall_performance.svg
[6]: https://serialized.net/source/vegeta_breaker.go
[7]: http://knowyourmeme.com/memes/its-over-9000

  