---
layout: tutorial
title: Getting Started with Apache Zeppelin
tutorial-id: 368
tutorial-series: Zeppelin
tutorial-version: hdp-2.5.0
intro-page: true
components: [ zeppelin, spark ]
---

Apache Zeppelin is a web-based notebook that enables interactive data analytics. With Zeppelin, you can make beautiful data-driven, interactive and collaborative documents with a rich set of pre-built language backends (or interpreters) such as Scala (with Apache Spark), Python (with Apache Spark), SparkSQL, Hive, Markdown, Angular, and Shell.

With a focus on Enterprise, Zeppelin has the following important features:

* Livy integration (REST interface for interacting with Spark)
* Security:
  * Execute jobs as authenticated user
  * Zeppelin authentication against LDAP
  * Notebook authorization

### **Prerequisites**

*   HDP 2.5
*   Spark 1.6.2

### **Launching Zeppelin**

If you haven't already, login to Ambari (operations console) using `maria_dev`/`maria_dev` as a username/password combination. Remember that Ambari is accessible on port 8080.

E.g. on a VirtualBox Sandbox, Ambari would be accessible at http://127.0.0.1:8080.

Note: If you're new to the HDP Sandbox environment, make sure to review [Learning the Ropes of the Hortonworks Sandbox](http://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/).

![scr1-login](https://raw.github.com/hortonworks/tutorials/hdp-2.5/assets/getting-started-with-apache-zeppelin/scr1-login.png)

Okay, once you're in Ambari just click the Views dropdown (upper-right corner) and select Zeppelin.

![scr2-views](https://raw.github.com/hortonworks/tutorials/hdp-2.5/assets/getting-started-with-apache-zeppelin/scr2-views.png)

Voila, you should see default Zeppelin menu with a list of demos and labs that you can run to explore great examples to get you quickly up and running.

![scr3-zeppelin-main](https://raw.github.com/hortonworks/tutorials/hdp-2.5/assets/getting-started-with-apache-zeppelin/scr3-zeppelin-main.png)

Now let's create your first notebook.

### **Creating a Notebook**

To create a notebook:

1. Under the “Notebook” tab, choose **+Create new note**.

2.  You will see the following window. Type a name for the new note (or accept the default): <br><br>![Screen Shot 2016-03-07 at 4.43.20 PM](http://hortonworks.com/wp-content/uploads/2016/03/Screen-Shot-2016-03-07-at-4.43.20-PM-300x112.png)

3.  Type sc.version into a paragraph in the note, and click the “Play” button (blue triangle): <br><br>![Screen Shot 2016-03-07 at 4.51.46 PM](http://hortonworks.com/wp-content/uploads/2016/03/Screen-Shot-2016-03-07-at-4.51.46-PM-300x21.png)<br>
SparkContext, SQLContext, ZeppelinContext will be created automatically. They will be exposed as variable names ‘sc’, ‘sqlContext’ and ‘z’, respectively, in scala and python environments.<br><br>
**Note:** The first run will take some time, because it is launching a new Spark job to run against YARN. Subsequent paragraphs will run much faster.<br><br>

4.  When finished, the status indicator on the right will say "FINISHED". The output should list the version of Spark in your cluster: <br>

~~~
res0: String = 1.6.2
~~~

### **Importing Notebooks**

Alternatively, instead of creating a new notebook, you may want to import an existing notebook instead.

There are two ways to import Zeppelin notebooks, either by pointing to json notebook file local to your environment or by providing a url to raw file hosted elsewhere, e.g. on github. We'll cover both ways of importing those files.

##### 1. Importing a JSON file

Click Import.

![scr4-import](https://raw.github.com/hortonworks/tutorials/hdp-2.5/assets/getting-started-with-apache-zeppelin/scr4-import.png)

Next, click "Choose a JSON here" button.

![src5-click-json](https://raw.github.com/hortonworks/tutorials/hdp-2.5/assets/getting-started-with-apache-zeppelin/scr5-click-json.png)

Finally, select the notebook you want to import.

![src6-choose-json](https://raw.github.com/hortonworks/tutorials/hdp-2.5/assets/getting-started-with-apache-zeppelin/scr6-choose-json.png)

Now you should see your imported notebook among other notebooks on the main Zeppelin screen.

##### 2. Importing a Notebook with a URL

Click Import.

![scr4-import](https://raw.github.com/hortonworks/tutorials/hdp-2.5/assets/getting-started-with-apache-zeppelin/scr4-import.png)

Next, click "Add from URL" button.

![src7-click-url](https://raw.github.com/hortonworks/tutorials/hdp-2.5/assets/getting-started-with-apache-zeppelin/scr7-click-url.png)

Finally, copy and paste the url to the (raw) json file.

![src8-import-url](https://raw.github.com/hortonworks/tutorials/hdp-2.5/assets/getting-started-with-apache-zeppelin/scr8-import-url.png)

Now you should see your imported notebook among other notebooks on the main Zeppelin screen.

### **Importing External Libraries**

As you explore Zeppelin you will probably want to use one or more external libraries. For example, to run [Magellan](http://hortonworks.com/blog/magellan-geospatial-analytics-in-spark/) you need to import its dependencies; you will need to include the Magellan library in your environment. There are three ways to include an external dependency in a Zeppelin notebook: **Using the %dep interpreter** (**Note**: This will only work for libraries that are published to Maven.)

<pre>%dep
z.load("group:artifact:version")
%spark
import ...</pre>

Here is an example that imports the dependency for Magellan:

<pre>%dep
z.addRepo("Spark Packages Repo").url("http://dl.bintray.com/spark-packages/maven")
z.load("com.esri.geometry:esri-geometry-api:1.2.1")
z.load("harsha2010:magellan:1.0.3-s_2.10")</pre>

For more information, see [https://zeppelin.incubator.apache.org/docs/interpreter/spark.html#dependencyloading](https://zeppelin.incubator.apache.org/docs/interpreter/spark.html#dependencyloading).