# Simple PHP Job Queue
Here's the basics (will fill out later......ha!)

## Install
```bash
composer require n0nag0n/simple-job-queue
```

## Usage#
### Adding a new job
#### MySQL
```php
<?php

use n0nag0n\Job_Queue

// default is mysql based job queue
$Job_Queue = new Job_Queue('mysql', [
	'mysql' => [
		'table_name' => 'new_table_name', // default is job_queue_jobs
		'use_compression' => false // default is true to use COMPRESS() and UNCOMPRESS() for payload
	]
]);

$PDO = new PDO('mysql:dbname=testdb;host=127.0.0.1', 'user', 'pass');
$Job_Queue->addDbConnection($PDO);

$Job_Queue->selectPipeline('send_important_emails');
$Job_Queue->addJob(json_encode([ 'something' => 'that', 'ends' => 'up', 'a' => 'string' ]));
```

#### SQLite3
```php
<?php

use n0nag0n\Job_Queue

// default is mysql based job queue
$Job_Queue = new Job_Queue('sqlite', [
	'sqlite' => [
		'table_name' => 'new_table_name' // default is job_queue_jobs
	]
]);

$PDO = new PDO('sqlite:example.db');
$Job_Queue->addDbConnection($PDO);

$Job_Queue->selectPipeline('send_important_emails');
$Job_Queue->addJob(json_encode([ 'something' => 'that', 'ends' => 'up', 'a' => 'string' ]));
```

### Running a worker
See `example_worker.php` for file or see below:
```php
<?php
	$Job_Queue = new Job_Queue('mysql');
	$Job_Queue->addDbConnection($f3->db);
	$Job_Queue = $f3->job_queue;
	$Job_Queue->watchPipeline('some_cool_pipeline_name');
	while(true) {
		$job = $Job_Queue->getNextJob();
		if(empty($job)) {
			// adjust to whatever makes you sleep better at night
			usleep(500000);
			continue;
		}

		echo "Processing {$job['id']}\n";
		$payload = json_decode($job['payload'], true);
		$new_payload = $payload;
		unset($new_payload['webhook_url']);

		try {
			$result = doSomethingThatDoesSomething($payload);

			if($result === true) {
				$Job_Queue->deleteJob($job);
			} else {
				$Job_Queue->buryJob($job);
			}
		} catch(Exception $e) {
			$Job_Queue->buryJob($job);
		}
	}
```

### Handling long processes
Supervisord is going to be your jam. Look up the many, many articles on how to implement this.