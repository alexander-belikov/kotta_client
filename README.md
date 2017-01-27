# REST Client for Cloud Kotta

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) [![Build Status](https://travis-ci.org/yadudoc/kotta_client.svg?branch=master)]

This is documentation for the REST client that can be used to manage jobs on the Cloud Kotta platform.
TuringCompute.net is an instance of the Cloud Kotta architecture hosted by the Knowledge Lab on AWS.

## Access to the Bastion Host

Once you have setup an account with Cloud Kotta shoot an email to yadunand@uchicago.edu with your
ssh public key to grant you a login account on bastion.turingcompute.net. This is where you'll be
using the REST client to manage jobs on Cloud Kotta.

## Getting started

In order to use the REST_client the only python package that needs to be installed is the requests package.
Here are the steps to get install dependencies and get the requisite code.

```bash
sudo pip install requests
git clone https://github.com/yadudoc/TuringClient
cd TuringClient
```

## Auth

With the security constraints associated with the data stored on the various storage systems, the REST
client requires you to have a valid access_token from logging in with amazon stored in a file. These
tokens generally expire in 1 hour. You can always do a relogin on turingcompute to get a URL with a
valid access_token. However this does limit scriptability/automation.

Here are the steps to get this temporary access_token:

1. Go to [Cloud Kotta](https://turingcompute.net/login)
2. Click Login with Amazon button, and login using credentials that have been registered with Cloud Kotta
   which will take you to the Login Success page.
3. Copy the complete url from your address bar from the Login Success page to a file.
4. Specify this file using the -a or --auth flag to the client when making requests that require user authentication.

* One potential solution to getting long living tokens is to use the authorization code grant method:
https://developer.amazon.com/public/apis/engage/login-with-amazon/docs/authorization_grants.html


## Requests

These are the different requests that are supported by the REST client:
* submit : Submit a job (auth required)
* status : Get the status and attributes of a specific job
* fetch  : Get all outputs generated by a job
* list   : List all jobs that have been submitted by the user (auth required)
* cancel : Cancel a job (auth required, pending implementation)

Here is a brief description of each of these different requests:

### Submit

To submit a job you must have a fresh auth token (less than an hour old) and a description of your
job in JSON. The JSON description is simple and captures the executable, inputs and outputs.
Inputs can be provided as a comma separated string in which each input is a URL. The URLs can
be http urls or S3 urls.

```json
{
  "user_email"         : "<EMAIL_ADDRESS_REGISTERED_WITH_TURING>",
  "executable"         : "/bin/echo 'Hello World!'",
  "jobname"            : "Hello World on Kotta",
  "queue"              : "Test",
  "script_name"        : "none.sh",
  "script"             : "", 
  "jobtype"            : "script",
  "output_file_stdout" : "STDOUT.txt",
  "output_file_stderr" : "STDERR.txt",
  "outputs"            : "foo.txt",
  "walltime"           : "5"
}
```

* [TODO] Add notes on Input URLS.

Here's how you'd submit a job that you've described in json

```bash
$ ./client.py -r submit -a ./auth.info -j <job_description.json>
# Sample output:
[Success] Job_id: dds3a31e-24de-44f0-1629-040e3330eb33
```


### Status

Each job that has been submitted successfully has a job_id associated with it
and you can use the job_id to request the status and attributes of any job.


```bash
$ ./client.py -r status -j dad3a7ae-91de-4af0-9a69-NOT-VALID
queue                 | Test
submit_stamp          | 2016-03-11 18:27:24
submit_time           | 2016-03-11 18:27:24
status                | completed
user_email            | yadunand@uchicago.edu
z_processing_dur      | 1.00426697731
walltime              | 300
jobtype               | script
z_stageout_dur        | 0.115742921829
username              | Yadu Nand B
job_id                | dad3a7ae-91de-4af0-9a69-NOT-VALID
executable            | /bin/echo test.sh
z_stagein_dur         | 0.134030103683
complete_time         | 2016-03-11 18:27:58
outputs               | foo.txt
outputs               | test.sh
outputs               | STDOUT.txt
outputs               | STDERR.txt
completed
```


### Fetch

In order to fetch all the outputs that have been generated by your job, you can
use the fetch request that will download all the results

```bash
$ ./client.py -r fetch -a auth.info -j dad3a7ae-91de-4af0-9a69-0c40ee00ebf1
Creating directory : dad3a7ae-91de-4af0-9a69-0c40ee00ebf1
File not available : foo.txt
Downloading file   : dad3a7ae-91de-4af0-9a69-0c40ee00ebf1/test.sh
Downloading file   : dad3a7ae-91de-4af0-9a69-0c40ee00ebf1/STDOUT.txt
Downloading file   : dad3a7ae-91de-4af0-9a69-0c40ee00ebf1/STDERR.txt
```


### List

To list all jobs you have submitted you can use the list request. Please note
that you need the auth file to make this request. Since no job_id is not meaningful
here use "all". 

```bash
$ ./client.py -r list -a auth.info -j all
JOBID                                   |STATUS         |JOBTYPE   |SUBMIT_STAMP
cf81ba18-cc41-11e5-a3b8-12d73991dalk    |pending        |script    |2016-02-05 19:51:13
b352e7b2-cad0-11e5-a3b8-12dassa73add    |active         |script    |2016-02-03 23:49:01
3c747e80-cc42-11e5-a3b8-12d73asss212    |completed      |script    |2016-02-05 19:54:15
eac59818-cbcd-11e5-a3b8-12d7112as3k7    |completed      |script    |2016-02-05 06:01:37
```