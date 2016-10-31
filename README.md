
The versions of Spark, Jupyter and Ipython at the moment of writing

__Spark: 2.0.0__
__Jupyter: 4.1.1__
__Ipython: 4.0.1__

### How to set up environment for Spark on Mac

 - Go to spark directory using command `cd <spark directory>`. For example, `cd spark-2.0.0-bin-hadoop2.7`
 
 -	Create soft link to point to `<spark directory>`, `ln -s <spark directory> <linked directory>`. Doing so avoids changing e    nvironmental variables or path in some cases like updating older version to new version. A change in the soft link makes everything easy.
 
 - (Create and) Edit `.profile` or `.bash_profile` using  `vi` or `nano`. In the profile, add path: `export SPARK_HOME=<soft linked spark directory>`
`export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin`

 - Return to terminal and input the command `~/.profile` or `source. profile` to make this work. Use `echo $PATH` or `echo $SPARK_HOME` to check.

 - Input command `pyspark` or `spark-shell` to check.

More in video: [https://www.youtube.com/watch?v=7AcStx0SXSo](https://www.youtube.com/watch?v=7AcStx0SXSo) or [http://blog.jobbole.com/86232/](http://blog.jobbole.com/86232/)


### How to integrate pyspark with Jupyter notebook

1. Start notebook from terminal by typing jupyter (ipython notebook)

      From what I have read so far, the method that works for ipython is 
      
      * first creating a ipython profile, `ipython profile create pyspark`;
      * add a startup file, `$ touch ~/.ipython/profile_spark/startup/00-spark-setup.py`;
      * add the following in startup file:

          ```
          import os
          import sys

          spark_home = os.environ.get('SPARK_HOME', None)
          sys.path.insert(0, os.path.join(spark_home, 'python'))
          sys.path.insert(0, os.path.join(spark_home, 'python/lib/py4j-0.8.2.1-src.zip'))
          execfile(os.path.join(spark_home, 'python/pyspark/shell.py'))
          ```

       * finaly start `ipython --profile=spark`.

      More details in [http://litaotao.github.io/ipython-notebook-spark?s=inner](http://litaotao.github.io/ipython-notebook-spark?s=inner), [http://blog.jobbole.com/86232](http://blog.jobbole.com/86232)

2. All of these just work for ipython. If you want it to work for jupyter, add more steps on the basis of previous ones.

      First make sure you understand the concept of kernelspecs in ipython: the kernels that ipython notebook uses at the moment of being started. Kernelspecs usually are folders and resides in `~/.ipython/kernels`. 

      ```
      $ mkdir -p ~/.ipython/kernels/spark # create a new kernelspec named spark
      $ touch ~/.ipython/kernels/spark/kernel.json # add kernel file
      {
          "display_name": "PySpark (Spark 1.5.2)", 
          "language": "python",
          "argv": [
              "/usr/bin/python2", # make sure the path is the one where your python2 exists
              "-m",
              "ipykernel",
              "--profile=spark"
              "-f",
              "{connection_file}"
          ]
      } 
      ```
      More in [http://thepowerofdata.io/configuring-jupyteripython-notebook-to-work-with-pyspark-1-4-0/](http://thepowerofdata.io/configuring-jupyteripython-notebook-to-work-with-pyspark-1-4-0/) or [http://limdauto.github.io/posts/2015-12-24-spark-ipython-notebook.html](http://limdauto.github.io/posts/2015-12-24-spark-ipython-notebook.html)
 
3. Directly add kernel spec in jupyter 

      Reference: [https://arnesund.com/2015/09/21/spark-cluster-on-openstack-with-multi-user-jupyter-notebook/](https://arnesund.com/2015/09/21/spark-cluster-on-openstack-with-multi-user-jupyter-notebook/) or [http://www.davidgreco.me/blog/2015/12/24/how-to-use-jupyter-with-spark-kernel-and-cloudera-hadoop-slash-spark](http://www.davidgreco.me/blog/2015/12/24/how-to-use-jupyter-with-spark-kernel-and-cloudera-hadoop-slash-spark)


      The shared jupyter kernelspec usually resides in `/usr/local/share/jupyter/kernels` but it is better to check it on jupyter website. Create a kernel file called kernel.json and add these in the file:

      ```
      {
       "display_name": "PySpark",
       "language": "python",
       "argv": [
        "/usr/bin/python2", # the path where your python2 exists
        "-m",
        "ipykernel",
        "-f",
        "{connection_file}"
       ],
       "env": {
        "SPARK_HOME": "<soft linked spark directory>", 
        "PYTHONPATH": "<soft linked spark directory>/python/:<soft linked spark directory>/python/lib/py4j-0.8.2.1-src.zip", # make sure your py4j version matches
        "PYTHONSTARTUP": "<soft linked spark directory>/python/pyspark/shell.py", # you can delete this line because whenever you start jupyter notebook, a spark context will be generated for you. If you want to customize the spark context object, delete this line. Otherwise keep it. 
        "PYSPARK_SUBMIT_ARGS": "--master <[ go to official website to check] (http://spark.apache.org/docs/latest/submitting-applications.html#master-urls)> pyspark-shell"
       }
      }
      ```

     **Remember not to open or edit json file using text editor.** That will cause unseen and irritating errors. See here: [jupyter kernelspec list]( https://github.com/jupyter/notebook/issues/1477). Use other good professional text editors. 

     All of these can also be achieved using terminal command:

     ```
     sudo mkdir -p /usr/local/share/jupyter/kernels/pyspark/
     cat <<EOF | sudo tee /usr/local/share/jupyter/kernels/pyspark/kernel.json
     {
      "display_name": "PySpark",
      "language": "python",
      "argv": [
       "/usr/bin/python2",
       "-m",
       "ipykernel",
       "-f",
       "{connection_file}"
      ],
      "env": {
       "SPARK_HOME": "/opt/spark/",
       "PYTHONPATH": "/opt/spark/python/:/opt/spark/python/lib/py4j-0.8.2.1-src.zip",
       "PYTHONSTARTUP": "/opt/spark/python/pyspark/shell.py",
       "PYSPARK_SUBMIT_ARGS": "--master spark://10.20.30.178:7077 pyspark-shell"
      }
     }
     EOF
     ```


4. Start from terminal using command pyspark

     *  Edit bash_profile using nano .bash_profile. 
     *	 Add the follwing in the file 
     
     ```
     export PATH=<soft linked directory>
     export PYSPARK_DRIVER_PYTHON=jupyter
     export PYSPARK_DRIVER_PYTHON_OPTS='notebook' pyspark
     ```
     *	Make these environment variables available by `source .profile`
     *	Check using command `pyspark`

    More in [http://stackoverflow.com/a/33065359](http://stackoverflow.com/a/33065359) or [https://www.youtube.com/watch?v=I5JtvpyM14U](ttps://www.youtube.com/watch?v=I5JtvpyM14U)

### Spark issues 
  
  - Pyspark: Exception: Java gateway process exited before sending the driver its port number
   
    [http://stackoverflow.com/a/36367669](http://stackoverflow.com/a/36367669) 
    Solution: There is a change in python/pyspark/java_gateway.py , which requires PYSPARK_SUBMIT_ARGS includes pyspark-shell if a PYSPARK_SUBMIT_ARGS variable is set by a user. `export PYSPARK_SUBMIT_ARGS="--master local[2] pyspark-shell" `

  - How to allocate more memory to executor memory in local mode

    [http://stackoverflow.com/questions/26562033/how-to-set-apache-spark-executor-memory](http://stackoverflow.com/questions/26562033/how-to-set-apache-spark-executor-memory)

   Solution: Set driver memory instead in two ways

      1. On Spark clients (systems from which you intend to launch Spark jobs), do the following:
      
         * Create <soft linked spark directory>/conf/spark-defaults.conf on the Spark client:
         
         `cp <soft linked spark directory>/conf/spark-defaults.conf.template /<soft linked spark directory>/conf/spark-defaults.conf`
         
         * Add the following to `<soft linked spark directory>/conf/spark-defaults.conf`:
          `spark.driver.memory=2g(or the amount you want to allocate)`.
         

      2. Start from terminal
     
       `$ ./bin/spark-shell(pyspark) --driver-memory 2g`




