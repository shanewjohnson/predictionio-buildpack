# [Heroku buildpack](https://devcenter.heroku.com/articles/buildpacks) for PredictionIO

Enables data scientists and developers to deploy custom machine learning services created with [PredictionIO](https://predictionio.incubator.apache.org).

This buildpack is part of an exploration into utilizing the [Heroku developer experience](https://www.heroku.com/dx) to simplify data science operations. When considering this proof-of-concept technology, please note its [current limitations](#user-content-limitations). We'd love to hear from you. [Open issues on this repo](https://github.com/heroku/predictionio-buildpack/issues) with feedback and questions.

## Releases

**July 31st, 2017**: merged a [breaking change](https://github.com/heroku/predictionio-buildpack/pull/44) to make the buildpack composable with Python, Node, and other Heroku buildpacks. **Going forward, this buildpack no longer supports PredictionIO 0.10.** If this does effect you, please checkout the details in the [original pull request](https://github.com/heroku/predictionio-buildpack/pull/44).

See [all releases](https://github.com/heroku/predictionio-buildpack/releases) with their changes.

## Engines

Create and deploy engines with **Scala 2.11.8**, **Spark 2.1.0**, & **Hadoop 2.7.3**.

Supported versions of PredictionIO:

* **0.12.0-SNAPSHOT**
    * with Scala 2.11, Spark 2.1, & Hadoop 2.7
    * pre-release version [built at commit](https://github.com/apache/incubator-predictionio/commit/bf84ede6fe475ec591e784eb453c6194befb8515)
      * Elasticsearch 5 client with HTTP basic authentication and connection pooling, supports [Bonsai add-on](https://elements.heroku.com/addons/bonsai)
      * Batch predictions via [`pio batchpredict`](https://github.com/apache/incubator-predictionio/blob/develop/docs/manual/source/batchpredict/index.html.md)
    * specify [`0.12.0-SNAPSHOT` in configs](CUSTOM.md#user-content-update-source-configs)
* **0.11.0-incubating**
    * with Scala 2.11, Spark 2.1, & Hadoop 2.7
    * specify [`0.11.0-incubating` in configs](CUSTOM.md#user-content-update-source-configs)
* ~~0.10.0-incubating~~
    * no longer supported on master
    * see how to [upgrade or temporarily fix](https://github.com/heroku/predictionio-buildpack/pull/44)

Get started with an engine:

* [Universal Recommender engine](https://github.com/heroku/predictionio-engine-ur)
  * presented at [TrailheaDX 2017](https://www.youtube.com/watch?v=MO0Bmty9fmc)
* [Classification engine](https://github.com/heroku/predictionio-engine-classification)
  * presented at [TrailheaDX 2017](https://www.youtube.com/watch?v=MO0Bmty9fmc) & [Dreamforce 2016](https://www.salesforce.com/video/297129/)
* [Template Gallery](https://predictionio.incubator.apache.org/gallery/template-gallery/)
  * starting-points for many use-cases
  * follow [custom engine docs](CUSTOM.md) to use with this buildpack

🐸 **[How to deploy an engine](CUSTOM.md)**

## Architecture

This buildpack transforms the [Scala](http://www.scala-lang.org) source-code of a PredictionIO engine into a [Heroku app](https://devcenter.heroku.com/articles/how-heroku-works).

![Diagram of Deployment to Heroku Common Runtime](http://marsikai.s3.amazonaws.com/predictionio-buildpack-arch-04.png)

The events data can be stored in:

* **PredictionIO event storage** backed by Heroku PostgreSQL
  * compatible with this buildpack's [built-in Data Flow features](DATA.md) providing initial data load & sync automation
  * compatible with most engine templates; required by some (e.g. [UR](https://github.com/heroku/predictionio-engine-ur))
  * supports RESTful ingestion & querying via PredictionIO's built-in [Eventserver](CUSTOM.md#user-content-eventserver)
* **custom data store** such as Heroku Connect with PostgreSQL or RDD/DataFrames stored in HDFS
  * requires a highly technical, custom implementation of `DataSource.scala`

## Limitations

### Memory

This buildpack automatically trains the predictive model during [release phase](https://devcenter.heroku.com/articles/release-phase), which executes Spark as a sub-process (i.e. [`--master local`](https://spark.apache.org/docs/2.1.0/#running-the-examples-and-shell)) within [one-off and web dynos](https://devcenter.heroku.com/articles/dynos). That dyno's memory capacity is a limiting factor at this time. Only [Performance dynos](https://www.heroku.com/pricing) with 2.5GB or 14GB RAM provide reasonable utility.

This limitation can be worked-around by pointing the engine at an existing Spark cluster. See: [customizing environment variables, `PIO_SPARK_OPTS` & `PIO_TRAIN_SPARK_OPTS`](CUSTOM.md#user-content-spark-configuration).

## Development & Testing

🛠 Follow the [local development](DEV.md) workflow to setup an engine on your computer.

🔍 See the [Data Flow docs](DATA.md) for how to leverage the built-in import & sync workflow.

🤓 Info for [testing](CUSTOM.md#user-content-testing) this buildpack & individual engines.
