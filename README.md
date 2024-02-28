# deltastreamer-on-glue


![1709042093711](https://github.com/soumilshah1995/deltastreamer-on-glue/assets/39345855/1fc0820b-9806-4769-895d-dc55aa3da9b5)


Step 1: Download Dataset and upload to S3


Link : https://drive.google.com/drive/folders/1BwNEK649hErbsWcYLZhqCWnaXFX3mIsg?usp=share_link



# Step 2: Download the Jar Files from maven or google drive 
```
jcommander-1.78.jar
hudi-spark3.3-bundle_2.12-0.14.0.jar
hudi-utilities-slim-bundle_2.12-0.14.0.jar
```

Download the jar from https://drive.google.com/drive/folders/1Rs9243i-D-jmFHPivlwdtODBRQpG1nys?usp=share_link
LInk :

# Step 3: Upload the code to Glue and select the Language to Scala 
```
import com.amazonaws.services.glue.GlueContext
import com.amazonaws.services.glue.MappingSpec
import com.amazonaws.services.glue.errors.CallSite
import com.amazonaws.services.glue.util.GlueArgParser
import com.amazonaws.services.glue.util.Job
import com.amazonaws.services.glue.util.JsonOptions
import org.apache.spark.SparkContext
import scala.collection.JavaConverters._
import org.apache.spark.sql.SparkSession
import org.apache.spark.api.java.JavaSparkContext
import org.apache.hudi.utilities.streamer.HoodieStreamer
import org.apache.hudi.utilities.streamer.SchedulerConfGenerator
import org.apache.hudi.utilities.UtilHelpers
import com.beust.jcommander.JCommander;
import com.beust.jcommander.Parameter;

object GlueApp {

  def main(sysArgs: Array[String]) {
    val args = GlueArgParser.getResolvedOptions(sysArgs, Seq("JOB_NAME").toArray)

    val BUCKET = "XXX"

    var config = Array(
      "--source-class", "org.apache.hudi.utilities.sources.ParquetDFSSource",
      "--source-ordering-field", "replicadmstimestamp",
      s"--target-base-path", s"s3://$BUCKET/testcases/",
      "--target-table", "invoice",
      "--table-type" , "COPY_ON_WRITE",
      "--hoodie-conf", "hoodie.datasource.write.keygenerator.class=org.apache.hudi.keygen.SimpleKeyGenerator",
      "--hoodie-conf", "hoodie.datasource.write.recordkey.field=invoiceid",
      "--hoodie-conf", "hoodie.datasource.write.partitionpath.field=destinationstate",
      s"--hoodie-conf", s"hoodie.streamer.source.dfs.root=s3://$BUCKET/test/",
      "--hoodie-conf", "hoodie.datasource.write.precombine.field=replicadmstimestamp"
    )
    
    val cfg = HoodieStreamer.getConfig(config)
    val additionalSparkConfigs = SchedulerConfGenerator.getSparkSchedulingConfigs(cfg)
    val jssc = UtilHelpers.buildSparkContext("delta-streamer-test", "jes", additionalSparkConfigs)
    val spark = jssc.sc

    val glueContext: GlueContext = new GlueContext(spark)
    Job.init(args("JOB_NAME"), glueContext, args.asJava)

    try {
      new HoodieStreamer(cfg, jssc).sync();
    } finally {
      jssc.stop();
    }

    Job.commit()
  }
}


```

![image](https://github.com/soumilshah1995/deltastreamer-on-glue/assets/39345855/ed55f500-62e1-42f4-ae54-276f18b7a8e5)

# Enjoy 


