Test Knime workflows from a Junit test.

[![Java CI with Maven](https://github.com/cooflydata/knime-testflow/actions/workflows/ci.yml/badge.svg)](https://github.com/cooflydata/knime-testflow/actions/workflows/ci.yml)
![Coverage](.github/badges/jacoco.svg)
[![DOI](https://zenodo.org/badge/doi/10.5281/zenodo.55805.svg)](http://dx.doi.org/10.5281/zenodo.55805)

The Knime Testing Framework can run a test workflow either:
* Inside Knime, if you right-click on a workflow in your local workspace, you can select "Run as workflow test".
* From the command line, using `knime -application org.knime.testing.NGTestflowRunner -root <workflow dir>`.

This repo gives you another option run a test workflow inside of a Junit @Test method declaration.

This project uses [Eclipse Tycho](https://www.eclipse.org/tycho/) to perform build steps.

# Usage

Using the plugin requires several steps.

## 1. Add repository

This plugin is available in the `https://cooflydata.github.io/updates` update site.

To make use of in a Tycho based project add to the `<repositories>` tag of the `pom.xml` file the following:
```
<repository>
    <id>cooflydata</id>
    <layout>p2</layout>
    <url>https://cooflydata.github.io/updates</url>
</repository>
```

## 2. Add dependency to tests

In the `Require-Bundle` attribute of the `META-INF/MANIFEST.MF` of the tests module add
```
com.github.cooflydata.knime.testing.plugin;bundle-version="[1.0.0,2.0.0)",
org.knime.testing;bundle-version="[4.0.0,6.0.0)",
```

## 3. Add test workflow

Create a test workflow as described in the ["Testing Framework" manual](https://github.com/knime/knime-core/blob/bf6f8c378694d5a435ef29cb469a7ced26ffca9f/org.knime.testing/doc/Regression%20Tests.pdf).

Place the workflow as a directory inside the `src/knime/` directory of the tests module.

## 4. Add test

Create a new test class and inside the class put the following:
```
@Rule
public ErrorCollector collector = new ErrorCollector();
private TestFlowRunner runner;

@Before
public void setUp() {
    TestrunConfiguration runConfiguration = new TestrunConfiguration();
    runner = new TestFlowRunner(collector, runConfiguration);
}

@Test
public void test_simple() throws IOException, InvalidSettingsException, CanceledExecutionException,
        UnsupportedWorkflowVersionException, LockFailedException, InterruptedException {
    File workflowDir = new File("src/knime/my-workflow-test");
    runner.runTestWorkflow(workflowDir);
}
```

This will test the workflow put in `src/knime/my-workflow-test` in the previous step.

This will run minimal checks, to check more configure `runConfiguration` object.  
For example add some more checks by adding 
```
runConfiguration.setTestDialogs(true);
runConfiguration.setReportDeprecatedNodes(true);
runConfiguration.setCheckMemoryLeaks(true);
```

## 5. Run tests

```
mvn verify
```

The test results can be found in the `T E S T S` section of the standard output.

## 6. Add GUI testing on GitHub actions.

As you might have noticed during the previouse step, running test will quickly show some dialogs and windows.
To show graphical user elements an X-server is required, sadly GitHub actions does not run an X-server.
A temporary X-server can be run with Xvfb, which is luckily available on all GitHub actions environments.

Prepend `xvfb-run` before the `mvn verify` command in the `.github/workflows/*.yml` file.

For example
```
script: xvfb-run mvn verify -B
```

# Build

```
mvn verify
```

An Eclipse update site will be made in `p2/target/repository/` repository.
The update site can be used to perform a local installation.
By default this will compile against KNIME AP v5.1, using the [KNIME-AP-5.1](targetPlatform/KNIME-AP-5.1.target) file.
To build instead for KNIME AP v4.7, use:
```
mvn verify -Dknime.version=4.7
```


# Development

Steps to get development environment setup based on https://github.com/knime/knime-sdk-setup#sdk-setup:

1. Install Java 17
2. Install Eclipse for [RCP and RAP developers](https://www.eclipse.org/downloads/packages/release/2018-12/r/eclipse-ide-rcp-and-rap-developers)
3. Configure Java 17 inside Eclipse Window > Preferences > Java > Installed JREs
4. Import this repo as an Existing Maven project
5. Activate target platform by going to Window > Preferences > Plug-in Development > Target Platform and check the `KNIME Analytics Platform (5.1) - com.github.cooflydata.knime.testing.targetplatform/KNIME-AP-5.1.target` target definition.

During import the Tycho Eclipse providers must be installed.

# New release

1. Update versions in pom files with `mvn org.eclipse.tycho:tycho-versions-plugin:set-version -DnewVersion=<version>-SNAPSHOT` command.
2. Commit and push changes
3. Create package with `mvn package`, will create update site in `p2/target/repository`
4. Append new release to an update site
  1. Make clone of an update site repo
  2. Append release to the update site with `mvn install -Dtarget.update.site=<path to update site>`
5. Commit and push changes in this repo and update site repo.

