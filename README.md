# Vert.x Service Factory

This is a `VerticleFactory` implementation which deploys a verticle given a service name.

The service name is used to lookup a JSON descriptor file which determines the actual verticle that is to be deployed,
and can also contain deployment options for the verticle such as whether it should be run as a worker, and default
config for the service.

It is useful as it decouples the service user from the actual verticle that is deployed and it allows the service
to provide default deployment options and configuration.

## Service name

The service name should be unique and the convention is that it is composed of three parts:

* The owner. This will usually be your organisation domain, e.g. `com.mycompany`
* The service name. The name of the service, e.g. `clever-db-service`
* The version. e.g. `1.0`

The service name is composed of the above three parts concatenated with `:`, e.g. `com.mycompany:clever-db-service:1.0`

The observant of you may have noticed that this looks just like a Maven co-ordinate (groupID + artifactID + version) -
no coincidence there. They have been chosen to be isomorphic so we can easily lookup services from Maven repositories.

## Usage

When deploying the verticle use the prefix `service`.

The verticle can be deployed programmatically like this:

    vertx.deployVerticle("service:com.mycompany:clever-db-service:1.0", ...)

Or can be deployed on the command line with:

    vertx run maven:com.mycompany:clever-db-service:1.0
    
Vert.x picks up `VerticleFactory` implementations from the classpath, so you just need to make sure the`ServiceVerticleFactory`
 jar is on the classpath.
    
It will already be on the classpath if you are running `vertx` on the command using the full distribution.

If you are running embedded you can declare a Maven dependency to it in your pom.xml (or Gradle config):

    <dependency>
      <groupId>io.vertx</groupId>
      <artifactId>vertx-service-factory</artifactId>
      <version>3.0.0</version>
    </dependency>
    
You can also register `VerticleFactory` instances programmatically on your `Vertx` instance using the `registerVerticleFactory`
method.

## Service descriptor

When you ask to deploy a service, the service factory first looks for a descriptor file on the classpath.

The descriptor file name is given by the service name, where the `:` has been replaced by `.` (because `:` is a reserved
character on Windows) and has had `.json` appended to it.

E.g. `com.mycompany.clever-db-service.1.0.json`

The descriptor file is a text file which must contain a valid JSON object.

At minimum the JSON must provide a `main` field which determines the actual verticle that will be deployed, e.g.

    {
        "main": "com.mycompany.cleverdb.MainVerticle"
    }

or

    {
        "main": "app.js"
    }

The JSON can also provide an `options` field which maps exactly to a `DeploymentOptions` object.

    {
        "main": "com.mycompany.cleverdb.MainVerticle",
        "options": {
            "config" : {
              "foo": "bar"
            },
            "worker": true,
            "isolationGroup": "mygroup"
        }
    }
    
When deploying a service from a service descriptor, any fields that are specified in the descriptor, such as `worker`,
`isolationGroup`, etc cannot be overridden by any deployment options passed in at deployment time.

The exception is `config`. Any configuration passed in at deploy time will override any corresponding fields in any
config present in the descriptor file.
