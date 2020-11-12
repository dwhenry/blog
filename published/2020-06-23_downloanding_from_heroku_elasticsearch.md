Getting a snapshot downloading from Heroku Elasticsearch

There was recently a requirement to get a copy of a elasticsearch (ES) dump onto our local machines,
Due to the amount of data, and the way it was being indexed, it would have taken around 8 days to
regenerate the data locally. We could have improved the performance on this by re-writing the indexing
process to using the bulk endpoint, however as this code was untested, and we were in the process of
decommisioning it the download seemed like a better solution.

Heroku is normally pretty good at giving you access to your data (see the postgres snapshot download
facility in the CLI). This is not the case of the Elasticsearch data, you do have the facility to create
snapshots, but these are stored on a S3 cluster that you can't get access to, you can restore
the snapshots, but only to other Heroku instances, so how do you go about getting a download?

The solution it turns out is pretty easy, you need to setup an additional snapshot repository,
pointed at a s3 bucket that you have permission to download from. The below outlines the steps
to get this setup and then downloaded.

## 1. Get access to the Elastcisearch instance.

This is actually pretty easy. Log into heroku, access the elasticsearch addon and under 'Application'
there is a copy endpoint button.

--image 1--

This only gives you the URL and port, which isn't super useful as it is password protected. I was able to retrieve
the username (default elastic username) and password from the heroku settings, but this may vary depending on
your setup. Alternatively you can reset the password, be careful if you are doing this on a production
system as you easy lock you main application out.

## 2. Get a storage location

As I mention at the top of this article we will be using s3 to store the snapshot, so you will need a bucket
in the desired region (we will talk more about regions below), and I would suggest it should be password protected.

Once you have these details you are good to proceed to connecting the repository

## 3. Setting up teh repository

This can be done using a PUT request to the ES system

```
URL:
<ES_HOST>/_snapshot/<REPOSITORY_NAME>?verify=false&pretty

BODY:
{
  "type": "s3",
  "settings": {
    "bucket": <BUCKET_NAME>,
    "region": <REGION>,
    "access_key": <ACCESS_KEY>,
    "secret_key": <SECRET_KEY>,
    "compress": "true",
    "base_path": "es_snapshots/"
  }
}
```

This works great is your bucket is in `us-east-1` which is teh default, but what is you need to store
the data in a different region? While you can set the region in the above PUT, you can only set the AWS
URL in the client settings in the  `elastcisearch.yaml` file
on the server - that we don't have access to and can't change. Reading the docs shows that this is actually
set at a [client level](https://www.elastic.co/guide/en/elasticsearch/plugins/current/repository-s3-client.html),
and due to data laws if you are running in you ES cluster in Europe then any backups must also be stored in
Europe, so it must be possible to connect to a eu-west-1 bucket. But how?

I am using a tool call `Elasticvue` to manage my ES indicies, using this I can connect to the ES host using the
details we got above. In the repository screen I could see the existing repository and but hovering over the settings
I was able to get the client they are using for `eu-west-1`.

So updating the body of the above PUT request to also set the client:

```
    "client:" <CLIENT_ID>
```

I can now connect to `eu-west-1` buckets.

## 4. Creating the snapshot

I used elasticvue to do this as well, it was simply a matter of clicking 'create snapshot' for the new repository and then waiting for it to finsih - about 10min.

## 5. Restoring the snapshot

You could do this by connecting to the same s3 bucket with your local ES instance and then doing a snapshot
restore, however I never managed to get this to work, and it actually turned out to be much easier to use
the AWS CLI to download the folder locally and then do a snapshot restore

```
aws s3 sync s3://<BUCKET_NAME>/ <LOCAL_PATH>
```

And that is it you should now have a copy of you ES indicies on your local machine.
