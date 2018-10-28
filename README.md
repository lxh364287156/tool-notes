Some notes of common tools
# kafka_job_client

### Introduction
- Make the work of processing message(`generic job` or `GdsJob`) more easier

### Get Started

#### API
Describe main functions and usage flow

- `Client` is a client wrapped kafka client with basic function: fetch and process
   messsage concurrently.
<pre>
package main

import (
	"math/rand"
	"git.garena.com/shopee/deep/kafka_job_client"
)

type userHandler struct{}

func (d *userHandler) GetMessageToJobHandle() kjc.MessageToJob {
	return func(message kafkaclient.Message, para kjc.Parameter) (interface{}, uint64, error) {
		return message, rand.Uint64(), nil
	}
}

func (d *userHandler) GetProcessHandle() kjc.Process {
	return func(job interface{}) error {
		return nil
	}
}

func buildConfig(conf string) *kjc.Config {
	var config *kjc.Config
	// you build logic here...
	return config
}

func main() {
	// server log init and other init...

	// build client config and set Handler.
	//
	config := buildConfig()
	config.Handler = &userHandler{}

	// NewClient
	client := kjc.NewClient(config)

	// Start client
	if err := client.Start(); err != nil {
		panic(err)
	}

	// Wait signal to stop server

	// Stop client
	client.Stop()
}
</pre>

- `GdsProcessor` used to manage process function of `GdsJob`.
<pre>
package main

import (
	"git.garena.com/shopee/deep/kafka_job_client"
)

// User's process func for GdsJob of different types
func function1(job *kjc.GdsJob, old, new interface{}) error { return nil }
func function1(job *kjc.GdsJob, old, new interface{}) error { return nil }

var (
	procUnits = []kjc.ProcUnit{
		{Table1, Command1, function1},
		{Table2, Command2, function2},
	}
)

// to implement your own RecordParser.
type recordParser struct {
}

func (r *recordParser) Parse(record *gds.GdsRecord) (result *kjc.ParsedRecord, err error) {
	// parse GdsRecord into ParsedRecord
	return
}

func main() {
	// init log and other functions.

	// init processor
	option := &kjc.GdsProcessorOption{
		ProcUnits: procUnits,
		// can also use NewGdsHandler to new GdsJob default parser.
		RecordParser: &recordParser{},
	}
	processor := kjc.NewGdsProcessor(option)

	// generate your job
	job := kjc.GdsJob{}

	// process your job
	processor.ProcessRecord(job.(*kjc.GdsJob))
}
</pre>

#### examples
- We provide two examples of services under examples diretory.
  One is the service to handle gds job, the other is the service for generic job.
- Usage of two services please refer to their README.

#### tools
- Producer is a tool to produce message for kafka.
- Currently supporting two types of data: `generic` and `gds` type.
  `generic` is for `generic_client`, `gds` is for `gdsjob_client`
