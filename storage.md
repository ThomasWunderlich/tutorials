# Storage
One of the most powerful spiral components, the storage component is designed to simplify work with remote files. Currently, the storage component supports:
Local server (harddrive), remote FTP, Amazon S3, Rackspace Files.

## How Storage Engine works
Storage component provides a high level set of operations used to manipulate location of `StorageObject` instances.

The general idea is to encode information about the file location in its own URI (in future `address`) which can either be a filename with string prefix or a valid URL. This methodology allows the developer (you, i assume) to skip steps related to detecting of file location and makes additional
columns in database unnecessary.

## Configuration
The configuration of storage components includes two main sections: `servers` and `containers`. Where server represent the local or remote location and protocol, container is used to provide an abstraction layer on top of the server.

### Common Container Options
Every container configuration should always include the following fields:

TABLE
server
prefix
options


### Local Server
Local server does not need any special configuration and mainly declared by class to be used.

example

Container connected to local server should include additional options like folder where files will be stored in and file permissions (by default `FileManager::RUNTIME`).

table

example

### FTP Server

Declaration of ftp container is very similar to local one and includes identical set of options.

copy from file 

> Make sure FTP user has permissions to write and change file mode.

### Amazon S3 Server

### Rackspace Server


## Create new StorageObject

## Operations

### Rename

### Replace

### Delete

### Get Filesize

### Get

## Switching storage servers


## Errata
In some very specific cases you have to use additional local container to prevent performance issues. 

```php
$file = Storage::create('local', '{basename}', $filename);

//Some file content manupulations
$image = Image::open($file->getFilename()); //Local file will be accessed
$image->resize(800, 600);
$image->save(); //Saved in local file

$file->replace('assets'); //"Permanent" storage
```

Generally speaking, you have to use local container if you want to perform some file operations (image resize, file reading) before storing file in "permanent" location. Skipping local container step will make system to upload and then download file from remote location (if such specified in config). I recommend always have defined "local" container connected to some temporary folder.

## Custom storage server
If you plan to create your own storage server with custom logic inside (like GridFS, non supported cloud platform) you only have to implement `Spiral\Components\Storage\ServerInterface` and register server instance in storage config.

> Spiral storage component was written originally in 2009-2010, it does not use official Amazon and Rackspace APIs. This problem will be solved in future framework releases. 
