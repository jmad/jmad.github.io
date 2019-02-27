[![Latest release](https://img.shields.io/github/release/jmad/jmad-core.svg?maxAge=1000)](https://github.com/jmad/jmad-core/releases)
[![Build Status](https://travis-ci.com/jmad/jmad-core.svg?branch=master)](https://travis-ci.com/jmad/jmad-core)
![License](https://img.shields.io/github/license/jmad/jmad-core.svg)
[![codecov](https://codecov.io/gh/jmad/jmad-core/branch/master/graph/badge.svg)](https://codecov.io/gh/jmad/jmad-core)
[![Codacy code quality](https://api.codacy.com/project/badge/Grade/b830f8eafc0441199d126967bd87d08c)](https://www.codacy.com/app/jmad/jmad-core?utm_source=github.com&utm_medium=referral&utm_content=jmad/jmad-core&utm_campaign=Badge_Grade)

# JMad

JMad is a Java API for the [MadX](http://mad.web.cern.ch/mad/) software, which is used at [CERN](http://www.cern.ch) and 
in many other accelerator labs to simulate particle accelerators. 
JMad plays a key role in CERNs accelerator control system as it is bridges numerical simulations with the control 
system of the real CERN accelerators. 

## Getting Started

In order to use JMad in your java projects add the following to your build files:

Maven:
```xml
<dependency>
    <groupId>jmad</groupId>
    <artifactId>jmad-core</artifactId>
    <version>X.Y.Z</version>
</dependency>
```

Gradle:
```groovy
compile 'jmad:jmad-core:X.Y.Z'
```

__IMPORTANT NOTE__
> The groupId of JMad (```jmad```) will change soon! It will become ```io.jmad``` for new versions. 
Stay tuned!

### Using a model

JMad comes with a concept called 'Model Definitions'. A model definition
specifies which madx files have to be loaded for a certain accelerator 
and certain optics (for madx users: sequence files, strength files, etc...).
Model definitions can be created programmatically or be loaded from different 
sources: zip files, git repos or jars in the classpath. For this example,
we will use the latter one. 

jmad-core comes with one model definition built in in its jar itself. To get a new 
instance of a model using this model definition we have to do the following:

```java
/* create a new JMad service */
JMadService jmadService = JMadServiceFactory.createJMadService();

/* then find a model definition */
JMadModelDefinition modelDefinition = jmadService.getModelDefinitionManager()
                                                 .getModelDefinition("example");

/* create the model and initialize it */
JMadModel model = jmadService.createModel(modelDefinition);
model.init();
``` 

After this we can use the model. For example we can retrieve all the elements the
sequence range:

```java
/* get all the elements */
List<Element> elements = model.getActiveRange().getElements();

/* print name and type of each element */
for (Element element : elements) {
    System.out.println("name: " + element.getName() + "; type: " + JMadElementType.fromElement(element));
}
```

... we can retrieve the actual optics, change some quadrupole and recalculate the optics:

```java
Range activeRange = model.getActiveRange();

/* retrieve an element by name (MONITOR) */
Monitor monitor = (Monitor) activeRange.getElement("BPMIH.22604");

/* retrieve the actual optics */
Optic optic = model.getOptics();

/* retrieve some optics values */
List<Double> betaxValues = optic.getValues(MadxTwissVariable.BETX, activeRange.getElements());

/* retrieve optics values for one element */
OpticPoint opticPoint = optic.getPoint(monitor);
double monX = opticPoint.getX();

/* change a quad strength by 10 percent */
Quadrupole aQuad = (Quadrupole) activeRange.getElement("MQIF.20400");
aQuad.setK1(aQuad.getK1() * 1.1);

/* IMPORTANT: refetch the optic since it has changed! */
optic = model.getOptics();
```

... or we can perform a custom twiss:

```java
TfsResultRequestImpl request = new TfsResultRequestImpl();

/* a regexp for the elements we want to retrieve */
request.addElementFilter("BPM.*");

/* and the variables, which we want to get */
request.addVariable(MadxTwissVariable.NAME);
request.addVariable(MadxTwissVariable.X);
request.addVariable(MadxTwissVariable.Y);

/* run the twiss and get the results */
TfsResult result = model.twiss(request);

List<String> elementNames = result.getStringData(MadxTwissVariable.NAME);
List<Double> xValues = result.getDoubleData(MadxTwissVariable.X);
List<Double> yValues = result.getDoubleData(MadxTwissVariable.Y);

/* print the values */
for (String name : elementNames) {
    int index = result.getElementIndex(name);
    System.out.println(name + ": X=" + xValues.get(index) + "; Y=" + yValues.get(index) + ";");
}
```

To close a model we have to call:
```java
model.cleanup();
```

## Additional Components

* There is also a [Swing GUI](https://github.com/jmad/jmad-gui) available, which
runs on top of jmad and allows to interactively change model parameters, plot
optics functions and can be embedded easily in other GUIs.
* To Retrieve model definitions from git repositories, the project 
[jmad-modelpack-service](https://github.com/jmad/jmad-modelpack-service)
can be used (works currently only with gitlab repos)
* [pyjmad](https://github.com/pymad/pyjmad) is a wrapper around jmad with which
all the features of jmad (including model definitions, GUI and retrieval from git repos) 
can be used as library in python.

## Further Reading

* An overview of jmad was given in a [paper at IPAC 2010](http://accelconf.web.cern.ch/AccelConf/IPAC10/papers/MOPEC006.pdf).
* A summary - including the GUI parts - can also be found in the [Phd Thesis of K. Fuchsberger](http://cds.cern.ch/record/1377386/files/CERN-THESIS-2011-075.pdf) (see page 61).  

