##AntMeter##
####解决了什么问题？####
1.假如你有3个场景需要测试，每个场景又需要跑20，50，100，150，200并发这5种场景。那么你可能需要一整天呆在电脑前，看着Jmeter运行然后结束，修改参数然后运行下一并发。3＊5=15种组合，花费相当多的时间和精力。用这个你就**解放**出来了！
2.每次运行一种场景，需要保存跑出来的结果。浪费时间不说，还容易记录错误。通常还需要记录下本次结果运行的时间，方便查NOAH。而用这个你又**解放**出来了！
3.运行结果的详细结果，UI界面是默认是没有保存下来的。而用这个东东你又**解放**了！。

####使用本工具，要有什么支持？####
* JMeter
* Ant
* JDK

####实现的原理####
通过官方提供的ant-jmeter包，配合ant来执行jmeter脚本(.jmx),使用ant-contrib来控制循环的执行所有的脚本。

####使用方法####
相关目录：
>jmxs：这个里面就是存放，你要运行的所有脚本了。

>libs: 这个里面放了3个工具所要使用的依赖包。

>build.xml： 这个就是ant脚本了。里面配置了所有运行所需的配置。

放入，你要执行的.jmx脚本到jmxs目录内。
在build.xml所在目录内，执行`ant`命令。
执行后，你会发现多了一个results目录，这里就是要存储结果的地方。results/jtl存放的就是运行的详细结果了。results/html就是咱们想要的汇总结果了。

####build.xml解释####

	<?xml version="1.0" encoding="utf-8"?>
	<project default="all">

	<!-- Define your Jmeter Home & Your Report Title & Interval Time Between Test-->
	<property name="report.title" value="WebLoad Test Report"/>
	<property name="jmeter-home" location="E:/apache-jmeter-2.8/" />
	<property name = "interval-time-in-seconds" value ="240"/>
	<!-- default path config, you can modify for your own requirement;Generally, you do not need to modify -->
	<property environment="env" />
	<property name="runremote" value="false"/>
	<property name="results.jtl" value="results/jtl"/>
	<property name="results.html" value ="results/html"/>
	<property name="jmxs.dir"  value= "jmxs"/>
	<!--  Diffrent version of Jmeter has its own ant-jmeter.jar,Please input the right versioin -->
	<path id="ant.jmeter.classpath">
		<pathelement location="${jmeter-home}/extras/ant-jmeter-1.1.1.jar" />
	</path>
	<taskdef name="jmeter"
		classname="org.programmerplanet.ant.taskdefs.jmeter.JMeterTask"
		classpathref="ant.jmeter.classpath" />
	<!-- just to support foreach by ant -->
	<taskdef resource="net/sf/antcontrib/antcontrib.properties" >
	    <classpath>
		<pathelement location="./libs/ant-contrib-20020829.jar" />
	   </classpath>
	</taskdef>

	<!-- use this config to generate html report; if not, may not display Min/Max Time in html-->
 	<path id="xslt.classpath">
        	<fileset dir="./libs" includes="xalan-2.7.1.jar"/>
        	<fileset dir="./libs" includes="serializer-2.9.1.jar"/>
    	</path>

	<target name="clean">
		<delete dir="results" />
		<delete file="jmeter.log" />
		<mkdir dir="${results.jtl}" />
		<mkdir dir="${results.html}" />
	</target>

	<target name="all-test" depends="clean">
		<foreach  param="jmxfile" target="test" >
		    <fileset dir="${jmxs.dir}">
		        <include name="*.jmx" />
		    </fileset>
		</foreach>
	</target>

	<target name="test" >
		<basename property="jmx.filename" file="${jmxfile}" suffix=".jmx"/>
		<echo message="---------- Processing ${jmxfile}  -----------"/>
		<jmeter jmeterhome="${jmeter-home}" resultlogdir="${results.jtl}" runremote="${runremote}" resultlog="${jmx.filename}.jtl"
		  testplan="${jmxs.dir}/${jmx.filename}.jmx">
		  <!--
			<testplans dir="${jmxs.dir}" includes="${jmx.filename}.jmx" />
			-->
			<jvmarg value="-Xincgc"/>
			<jvmarg value="-Xms256m"/>
			<jvmarg value="-Xmx512m"/>
		</jmeter>
		<sleep seconds="60"></sleep>
		<!--Generate html report-->
		<tstamp><format property="report.datestamp" pattern="yyyy-MM-dd HH:mm:ss"/></tstamp>
		<xslt  in="${results.jtl}/${jmx.filename}.jtl"  out="${results.html}/${jmx.filename}.html"  classpathref="xslt.classpath"
			style="${jmeter-home}/extras/jmeter-results-report_21.xsl" >
		<param name="dateReport" expression="${report.datestamp}"/>
		<param name="showData" expression="n"/>
		<param name="titleReport" expression="${report.title}:[${jmx.filename}]"/>
		</xslt>

		<echo message="Sleep ${interval-time-in-seconds} Seconds, and then start next Test; Please waiting ......"/>
	   	<sleep seconds="${interval-time-in-seconds}"></sleep>
	</target>

	<target name="copy-images" depends="all-test">
        	<copy file="${jmeter-home}/extras/expand.png" tofile="${results.html}/expand.png"/>
        	<copy file="${jmeter-home}/extras/collapse.png" tofile="${results.html}/collapse.png"/>
    	</target>

	<target name="all" depends="all-test, copy-images" />
</project>

自己用的时候，主要关注这样几个点就行了。
`<property name="report.title" value="WebLoad Test Report"/>`
定义，你的报告名称。
`<property name="jmeter-home" location="E:/apache-jmeter-2.8/" />`
指定你的jmeter home在哪儿。
`<property name = "interval-time-in-seconds" value ="240"/>`
这个是指定，不同场景运行之间的间隔，也就是说不能一直压机器嘛，运行一个场景后，让服务器休息下，同时也方便后期查看noah信息不是？
`<property name="runremote" value="false"/>`
此处用于指定是否要分布式运行jmeter，当然这个支持分布式运行需要你在jmeter.properties进行配置好的了。

####工具目前已知问题####
在并发量太大的时候，有可能存在，运行一个jmx后，无法finish的情况。从而导致无法执行下一个jmx。

####扩展####
既然本地都可以用ant跑了，放到jenkins/hudson上也是OK的。
