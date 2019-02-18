# Cloud Foundry Ext-Lib Buildpack

CBX Extended Library buildpack for injecting deployment time 3rd party licensed libs into Cloud Foundry droplets.

## Internals

The buildpack detects if the variable `CBX_EXTLIB_LOCATIONS` has been set - no harm done if not set or empty, buildpack will exit gracefully allowing the next buildpack to complete.

For each URI in specified in the variable `CBX_EXTLIB_LOCATIONS` (JSON Array). the buildpack will attempt to download the referenced file to the `deps/ext_lib` directory. Failure of this download will result into unsuccessful execution of the buildpack, hence breaking the deployment.

## Example app

An example application has been provided in [this repository](https://bitbucket.org/igtbdigital/sb-classpath-client/src/master/).

A simple `cf push` would deploy the application below where the `ext-lib-buildpack` would download the required dependencies specified as a JSON array in `CBX_EXTLIB_LOCATIONS` to the `deps/ext_libs` directory in the droplet.

Note the `JAVA_OPTS` setting to point the Spring Boot `PropertiesLoader` to the additional lib directory as described [here](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/loader/PropertiesLauncher.html).

~~~yaml
applications:
  - name: sb-classpath-client
    instances: 1
    buildpacks: 
      - https://github.com/rdc-lda/ext-libs-buildpack.git
      - java_buildpack
    path: target/sb-classpath-client-0.0.1-SNAPSHOT.jar
    memory: 1024M
    disk_quota: 256M
    routes:
      - route: rh-sb-classpath-client.cfapps.io
    stack: cflinuxfs2
    env:
      # Test logging
      enable.testLog: false
      testLog.freq: 5000
      # Classpath file loading
      enable.classpath.loading: true
      classpath.loadFile: classpath-file.txt
      JAVA_OPTS: '-Dloader.path=$PWD/../deps/ext_libs'
      CBX_EXTLIB_LOCATIONS: '[ "https://s3.amazonaws.com/cbx-ext-libs/classpath-file.txt" ]'
~~~