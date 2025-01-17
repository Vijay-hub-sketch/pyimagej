# Common Errors

## Error in "mvn.CMD -B -f pom.xml" dependency:resolve: 1

This indicates a problem running Maven on your system and will require more
debugging effort. Please post [on the
forum](https://forum.image.sc/tag/pyimagej) and include either:

* The results of manually running the Maven command with an added `-X` flag: `path\to\mvn.CMD -B -f -X path\to\pom.xml`
* The results of re-running the same `imagej.init` call after:
   * Deleting your `~/.jgo` directory
   * Adding `import logging` and `logging.basicConfig(level = logging.DEBUG)` to the top of your script

### Could not transfer artifact

If the debugging output includes notices such as:
```
DEBUG:jgo: [ERROR] Non-resolvable import POM: Could not transfer artifact net.imglib2:imglib2-imglyb:pom:1.0.1 from/to scijava.public (https://maven.scijava.org/content/groups/public): Transfer failed for https://maven.scijava.org/content/groups/public/net/imglib2/imglib2-imglyb/1.0.1/imglib2-imglyb-1.0.1.pom @ line 8, column 29: Connect to maven.scijava.org:443 [maven.scijava.org/144.92.48.199] failed: Connection timed out:
```
This suggests you may be behind a firewall that is preventing Maven from downloading the necessary components. In this case you have a few options to try:
1. Configure your proxy settings directly in your python code (replacing `myproxy.domain` and port `8080` as appropriate)
   ```
   import scyjava
   System = scyjava.jimport('java.lang.System')
   mydomain = "myproxy.domain"
   myport = "8080"
   System.setProperty("http.proxyHost", mydomain)
   System.setProperty("http.proxyPort", myport)
   System.setProperty("https.proxyHost", mydomain)
   System.setProperty("https.proxyPort", myport)
   ```
2. Configure your proxy settings [through Maven](https://www.baeldung.com/maven-behind-proxy) in the `<settings>..</settings>` block of your `$HOME\.m2\settings.xml` file
   ```
   <proxies>
     <proxy>
       <id>Your company proxy</id>
       <active>true</active>
       <protocol>https</protocol>
       <host>proxy.mycompany.com</host>
       <port>8080</port>
     </proxy>
   </proxies>
   ```
3. Initialize with a local `Fiji.app` installation. In this case you will also have to manually download the latest `.jar` files for [imglib2-unsafe](https://maven.scijava.org/#nexus-search;quick~imglib2-unsafe) and [imglib2-imglyb](https://maven.scijava.org/#nexus-search;quick~imglib2-imglyb) and place them in your local `Fiji.app/jars` directory, as these are required for PyImageJ but not part of the standard Fiji distribution.

### Unable to find valid certification path

If the debugging output includes notices such as:
```
Caused by: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
    at sun.security.validator.PKIXValidator.doBuild (PKIXValidator.java:397)
    at sun.security.validator.PKIXValidator.engineValidate (PKIXValidator.java:240)
```
This suggests the version of Java being used is too old and contains outdated certificate information. This behavior has been confirmed with the `openjdk` installed from the default conda channel (i.e. `conda install openjdk`). Try using an openjdk from the [conda-forge channel](https://anaconda.org/conda-forge/openjdk) instead.

## I ran a plugin and see an updated image, but the numpy array and dataset are unchanged.

This bug can occur in certain circumstances when using original ImageJ plugins
which update a corresponding `ImagePlus`. It can be worked around by calling:

```python
imp = ij.py.WindowManager.getCurrentImage()
ij.py.synchronize_ij1_to_ij2(imp)
```

## The GUI has issues on macOS

The default UI may not work on macOS. You can try using ImageJ's
Swing-UI-based GUI instead by initializing with:

```python
import imagej
ij = imagej.init(..., headless=False)
ij.ui().showUI("swing")
```

Replacing `...` with one of the usual possibilities
(see [Initialization.md](Initialization.md)).

See [this thread](https://github.com/imagej/pyimagej/issues/23)
for additional information and updates.

## Original ImageJ classes not found

If you try to load an original ImageJ class (with package prefix `ij`),
and get a `JavaException: Class not found` error, this is because
the environment was initialized without the original ImageJ included.
See [Initialization.md](Initialization.md).

## Not enough memory

You can increase the memory available to the JVM before starting it.
See [Initialization.md](Initialization.md).

## log4j:WARN 

PyImageJ does not currently ship a log4j implementation, which results in an
obnoxious warning at startup:

```
log4j:WARN No appenders could be found for logger (org.bushe.swing.event.EventService).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
```

This can safely be ignored and will be addressed in a future patch.
