# ruby-jmeter-docker
A good reference for running load tests with ruby jmeter and docker. You can clone this repo, create your own scenario folder and load test script then package up your load test to run anywhere. Running on a multi-cpu instance, you should be able to generate lots of load, ymmv.

Conceptual clarification: jmeter is a multi-threaded load generation program. Each "v-user" is a thread. The "scenario" defines what the script is to do (how many threads, what requests to make, etc). ruby-jmeter is just a helper script that generates a valid jmeter scenario (an xml file) then shells-out to run jmeter itself.

## basic scenario
An example script has been provided in scenarios/basic/load.rb. It has a basic restful POST example with jmeter ramping up and sustaining a fixed throughput. This is a solid pattern for load testing. The script also includes the needed CLI flag handling. Just copy it, and create your own scenarios with https://github.com/flood-io/ruby-jmeter.

Tips:
- load balancers (e.g. AWS ALB/ELB) require warmup (ALB's with default 3 nodes can be stressed with merely 100's of rps). lb's only scale out "nodes" only after receiving increased load. Use --rampup to ensure you double load only every 5m or so.
- also: https://www.blazemeter.com/blog/dns-cache-manager-right-way-test-load-balanced-apps/
- You can also use the --rampup period to find your breaking point (again ramping up slowly)
- Once you've found the limit of your target system, run your final load test (e.g. testing peak load for an hour)

```
usage: scenarios/basic/load.rb [options]
    -u, --url         target url
    -t, --throughput  requests per second
    -d, --duration    duration in seconds
    -r, --rampup      rampup in seconds
    -h, --threads     threads, if you want to be specific
    -g, --gui         use gui (local only) be sure to export JMETER_BIN
    --help            
```

## Run with docker
```
docker build -t jmeter:latest .

docker run -it -v `pwd`:/mnt jmeter:latest \
scenarios/basic/load.rb --throughput 2 --duration 20 --rampup 1 --url https://www.mockbin.org/request
```

The -v option mounts your current dir, so you can inspect the output files from ruby-jmeter and jmeter
- jmeter.log (runtime info)
- jmeter.jtl csv file with actual results (one request per line)
- ruby-jmeter.jmx the XML file that ruby-jmeter creates as an input for jmeter itself
- html/index.html

### viewing the report
```
open output/html/index.html
```

## Local development

Here are some steps if you wish to use the load testing scripts locally

### Install dependencies

You will need openjdk 1.8, Ruby 2.2+, Jmeter (https://jmeter.apache.org/download_jmeter.cgi)

### Setup

```
gem install ruby-jmeter slop json
export JMETER_BIN=PATH_TO_JMETER_BIN
```

### Run
```
ruby scenarios/basic/load.rb -t 2 -d 20 -r 1 -u https://www.mockbin.org/request
``` 
When running locally, you might find jmeter's gui mode quite useful (the basic scenario has a --gui flag to enable that)

### Create an html report

```
$JMETER_BIN/jmeter -g jmeter.jtl -o output
open output/index.html
```

