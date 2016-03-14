JPMML-Android
=============

PMML evaluator library for the Android operating system (http://www.android.com/).

# Features #

* Full support for PMML specification versions 3.0 through 4.2. The evaluation is handled by the [JPMML-Evaluator] (https://github.com/jpmml/jpmml-evaluator) library.

# Prerequisites #

* Android 3.0 or newer.

# Installation #

JPMML-Android depends on the `1.2-SNAPSHOT` version of the JPMML-Evaluator library, which needs to be built and installed locally:
```
git clone https://github.com/jpmml/jpmml-evaluator.git
cd jpmml-evaluator
mvn clean install
```

After that, enter the project root directory and build using [Apache Maven] (http://maven.apache.org/):
```
mvn clean install
```

The build produces two files:
* `pmml-android/target/pmml-android-1.0-SNAPSHOT.jar` - Library JAR file.
* `pmml-android-example/target/pmml-android-example-1.0-SNAPSHOT.apk` - Example App APK file.

# Usage #

Android does not provide its own JAXB runtime, and is generally unsuitable for working with standard JAXB runtimes such as GlassFish Metro or EclipseLink MOXy. Therefore, the `org.dmg.pmml.PMML` instance has to be obtained by other means.

The suggested workaround is to transmit models using Java Serialization.

The conversion from PMML data format to Java Serialization (SER) data format takes place during APK build:
```
static
public void convertPmmlToSer(File pmmlFile, File serFile) throws Exception {
	PMML pmml;

	try(InputStream is = new FileInputStream(pmmlFile)){
		Source source = ImportFilter.apply(new InputSource(is));

		pmml = JAXBUtil.unmarshalPMML(source);
	}

	// Remove SAX Locator information
	LocatorNullifier locatorNullifier = new LocatorNullifier();
	locatorNullifier.applyTo(pmml);

	try(OutputStream os = new FileOutputStream(serFile)){
		SerializationUtil.serializePMML(pmml, os);
	}
}
```

The SER file should be placed into the assets directory.

The App can use the `org.jpmml.android.EvaluatorUtil#createEvaluator(InputStream)` utility method to create a model evaluator instance:
```
public ModelEvaluator<?> loadSer(String serName) throws Exception {
	AssetManager assetManager = getAssets();

	try(InputStream is = assetManager.open(serName)){
		return org.jpmml.android.EvaluatorUtil.createEvaluator(is);
	}
}
```

The Java serialization and deserialization work is handled by the `org.jpmml.model.SerializationUtil` utility class, which was introduced in JPMML-Model library version 1.2.12. If either the APK build or the App cannot find this class, then you are using an outdated version of the JPMML-Evaluator library.

# License #

JPMML-Android is licensed under the [GNU Affero General Public License (AGPL) version 3.0] (http://www.gnu.org/licenses/agpl-3.0.html). Other licenses are available on request.

# Additional information #

Please contact [info@openscoring.io] (mailto:info@openscoring.io)
