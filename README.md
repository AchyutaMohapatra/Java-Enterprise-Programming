J2EESample
========================

What is it?
-----------

This is your project! It's a sample, deployable Maven 3 project to help you
get your foot in the door developing with Java EE 6 on JBoss AS 7 or EAP 6. This 
project is setup to allow you to create a compliant Java EE 6 application 
using JSF 2.0, CDI 1.0, EJB 3.1, JPA 2.0 and Bean Validation 1.0. It includes
a persistence unit and some sample persistence and transaction code to help 
you get your feet wet with database access in enterprise Java. 

System requirements
-------------------

All you need to build this project is Java 6.0 (Java SDK 1.6) or better, Maven
3.0 or better.

The application this project produces is designed to be run on a JBoss AS 7 or EAP 6. 
The following instructions target JBoss AS 7, but they also apply to JBoss EAP 6.
 
With the prerequisites out of the way, you're ready to build and deploy.

Deploying the application
-------------------------
 
First you need to start JBoss AS 7 (or EAP 6). To do this, run
  
    $JBOSS_HOME/bin/standalone.sh
  
or if you are using windows
 
    $JBOSS_HOME/bin/standalone.bat

To deploy the application, you first need to produce the archive to deploy using
the following Maven goal:

    mvn package

You can now deploy the artifact to JBoss AS by executing the following command:

    mvn jboss-as:deploy

This will deploy `target/J2EESample.war`.
 
The application will be running at the following URL <http://localhost:8080/J2EESample/>.

To undeploy from JBoss AS, run this command:

    mvn jboss-as:undeploy

You can also start JBoss AS 7 and deploy the project using Eclipse. See the JBoss AS 7
Getting Started Guide for Developers for more information.
 
Running the Arquillian tests
============================

By default, tests are configured to be skipped. The reason is that the sample
test is an Arquillian test, which requires the use of a container. You can
activate this test by selecting one of the container configuration provided 
for JBoss AS 7 (remote).

To run the test in JBoss AS 7, first start a JBoss AS 7 instance. Then, run the
test goal with the following profile activated:

    mvn clean test -Parq-jbossas-remote

Importing the project into an IDE
=================================

If you created the project using the Maven archetype wizard in your IDE
(Eclipse, NetBeans or IntelliJ IDEA), then there is nothing to do. You should
already have an IDE project.

Detailed instructions for using Eclipse with JBoss AS 7 are provided in the 
JBoss AS 7 Getting Started Guide for Developers.

If you created the project from the commandline using archetype:generate, then
you need to import the project into your IDE. If you are using NetBeans 6.8 or
IntelliJ IDEA 9, then all you have to do is open the project as an existing
project. Both of these IDEs recognize Maven projects natively.

Downloading the sources and Javadocs
====================================

If you want to be able to debug into the source code or look at the Javadocs
of any library in the project, you can run either of the following two
commands to pull them into your local repository. The IDE should then detect
them.

    mvn dependency:sources
    mvn dependency:resolve -Dclassifier=javadoc
    
    Java EE 7 using Eclipse
=======================

Project creation
----------------

* Create a simple Dynamic Web Project
** Choose the module version as ``3.1''
** Add a new configuration for WildFly
** Add `index.html` in ``WebContent''
** ``Run as'', ``Run on Server''
** Change content on `index.html` and show http://docs.jboss.org/tools/whatsnew/livereload/livereload-news-1.0.0.Alpha2.html[LiveReload]

Servlet
-------

* Add a new Servlet
** In Servers tab, select the module, right-click and select ``Restart'' to restart the module
** Show http://localhost:8080/HelloJavaEE7/TestServlet

Persistence
-----------

* Right-click on project, select ``Properties'', search for ``facet'', enable ``JPA'', click on ``Apply''
** Talk about ``Further configuration available'' and default data source (no configuration required)
* Right-click on project, select ``New'', ``JPA Entity'' and give the name as `Student`
* Add a primary key in the wizard. Use `long` datatype and `id` name.
* Generate Getter/Setter by clicking ``Source'', ``Getters and Setters''.
* Add `toString` implementation
* The updated class should look like:
+
[source, java]
----
@Entity
@XmlRootElement
@NamedQuery(name="findAllStudents", query="select s from Student s")
public class Student implements Serializable {
	   
	@Id
	private long id;
	private static final long serialVersionUID = 1L;

	public Student() {
		super();
	}   
	
	public Student(long id) {
		this.id = id;
	}
	public long getId() {
		return this.id;
	}

	public void setId(long id) {
		this.id = id;
	}
	
	public String toString() {
		return "Student[" + id + "]";
	}
}
----
+
** `@XmlRootElement` in `Student` entity enables XML <-> Java conversion.
** `@NamedQuery(name="findAllStudents", query="select s from Student s")` in `Student` enables querying all students.
* Add the following properties to `persistence.xml`:
+
[source.xml]
----
<properties>
	<property name="javax.persistence.schema-generation.database.action"
		value="drop-and-create" />
	<property name="javax.persistence.schema-generation.create-source"
		value="metadata" />
	<property name="javax.persistence.schema-generation.drop-source"
		value="metadata" />
	<property name="hibernate.show_sql" value="true"/>
	<property name="hibernate.format_sql" value="true"/>		
</properties>
----
+
* Show ``server console'' when the application is deployed. Show the generated SQL statements.

Setup console to watch database
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Download
  https://github.com/jboss-developer/jboss-eap-quickstarts/blob/6.3.0.GA/h2-console/h2console.war?raw=true[H2 console war] to WildFly's `standalone/deployments` directory
+
[source,text]
----
curl -L https://github.com/jboss-developer/jboss-eap-quickstarts/blob/6.3.0.GA/h2-console/h2console.war?raw=true -o h2console.war
----
+
* Start the server and access http://localhost:8080/h2console
* Enter the JDBC URL and credentials. To access the "test" database your application uses, enter these details (everything else is default):
** JDBC URL: `jdbc:h2:mem:test;DB_CLOSE_ON_EXIT=FALSE;DB_CLOSE_DELAY=-1`
** User Name: `sa`
** Password: `sa`

RESTful Web Services
--------------------

* Create a JAX-RS resource `StudentEndpoint`
** `rest.RestApplication` is added and specifies the base URI for REST resoures as `/rest`.
** Use `Student` entity as the basis, select `create`, `findById`, `listAll` methods to be generated.
* Update the class so that it looks like as shown:
+
[source,java]
----
@RequestScoped
@Path("/students")
public class StudentEndpoint {
	
	@PersistenceContext EntityManager em; 

	@Transactional
	@POST
	@Consumes({ "application/xml", "application/json", "text/plain" })
	public void create(final long id) {
		Student student = new Student(id);
		em.persist(student);
	}

	@GET
	@Path("/{id:[0-9][0-9]*}")
	@Produces({ "application/xml", "application/json" })
	public Response findById(@PathParam("id") final Long id) {
		Student student = em.find(Student.class, id);
		if (student == null) {
			return Response.status(Status.NOT_FOUND).build();
		}
		return Response.ok(student).build();
	}

	@GET
	@Produces("application/xml")
	public Student[] listAll(
			@QueryParam("start") final Integer startPosition,
			@QueryParam("max") final Integer maxResult) {
		TypedQuery<Student> query = em.createNamedQuery("findAllStudents", Student.class);
		final List<Student> students = query.getResultList();
		return students.toArray(new Student[0]);
	}
}
----
+
* Use ``Advanced REST Client'' in Chrome
** Make a GET request to http://localhost:8080/HelloJavaEE7/rest/students
*** Add Accept: application/xml
*** Show empty response
** Make a POST request
*** Payload as `1`
*** ``Content-Type'' header to `text/plain`
** Make a GET request to validate the data is posted
* Edit `doGet` of `TestServlet` to match the code given below
+
[source,java]
----
ServletOutputStream out = response.getOutputStream();
out.print("Retrieving results ...");
Client client = ClientBuilder.newClient();
Student[] result = client
.target("http://localhost:8080/HelloJavaEE7/rest/students")
.request()
.get(Student[].class);
for (Student s : result) {
	out.print(s.toString());
}
----
* Access the Servlet in the browser to show the results and explain JAX-RS Client API


Bean Validation
---------------

* Change `create` method to add Bean Validation constraint
+
[source,java]
----
public void create(@Min(10) final long id) {
----
+
* Make a POST request with payload as `1` and show an error is being received


CDI
---

* Generate an interface `Greeting`
+
[source,java]
----
public interface Greeting {
	public String sayHello();
}
----
+
* Create a new class `SimpleGreeting`, implement the interface as:
+
[source, java]
----
public class SimpleGreeting implements Greeting {

	@Override
	public String sayHello() {
		return "Hello World";
	}

}
----
+
* Inject the bean in Servlet as `@Inject Greeting greeting;`
* Print the output as `response.getOutputStream().print(greeting.sayHello());`
* Show ``New missing/unsatisfied dependencies'' error and explain default injection
* Add `@Dependent` on bean
* Create a new class `FancyGreeting`, implement the interface, add `@Dependent`
* Create a new qualifier using ``New'', ``Qualifier Annotation Type''

Advanced CDI
------------

* Wizards:
http://docs.jboss.org/tools/4.1.x.Final/en/cdi_tools_reference_guide/html/chap-CDI_Tools_Reference_Guide-Creating_a_CDI_Web_Project.html[New CDI Web Project Wizard],
http://docs.jboss.org/tools/4.1.x.Final/en/cdi_tools_reference_guide/html/chap-CDI_Tools_Reference_Guide-Wizards_and_Dialogs.html#d0e555[CDI Wizards]
* Content assist: CDI Named Beans are available in JSF EL #{} content assist in XHTML/Java/XML files (See JSF)
* Validation:
http://docs.jboss.org/tools/4.1.x.Final/en/cdi_tools_reference_guide/html/chap-CDI_Tools_Reference_Guide-Validation.html
* Navigation (open the bean producer from the @Inject annotation for example):
http://docs.jboss.org/tools/4.1.x.Final/en/cdi_tools_reference_guide/html/chap-CDI_Tools_Reference_Guide-Hyperlink_Navigation.html[Java source navigation], from EL #{} to CDI bean (See JSF)
* Open CDI Named bean: http://docs.jboss.org/tools/4.1.x.Final/en/cdi_tools_reference_guide/html_single/index.html#d0e597
* Beans.xml editor: Content assist, Navigation, Validation
http://docs.jboss.org/tools/whatsnew/cdi/cdi-news-3.2.0.Beta1.html
* Search usage: https://issues.jboss.org/browse/JBIDE-8705[Injection Points], EL #{} (See JSF)
* CDI 1.2 support was introduced in JBoss Tools 4.3.0.Alpha1: http://tools.jboss.org/documentation/whatsnew/jbosstools/4.3.0.Alpha1.html#cdi
But it's mostly about showing CDI 1.2 as available in our wizards (New CDI Project wizard for example) + minor bug fixing. In JBoss Tools 4.2 (Eclipse Luna) you can use CDI 1.1 in wizards for CDI 1.2 projects since 1.2 is just a maintenance release and CDI Tools relays on actual CDI jars from the project's class path. 

Batch
-----

* Open https://github.com/javaee-samples/javaee7-samples/[Java EE 7 Samples project] and show Job XML
* http://tools.jboss.org/documentation/whatsnew/jbosstools/4.3.0.Alpha1.html#batch[Batch job XML editor] - available in JBoss Tools 4.3.0.Alpha1
* Upcoming 4.3.0.Alpha2 features: Validation, Content Assist, Navigation, - https://issues.jboss.org/browse/JBIDE-18857

JavaServer Faces
----------------

* EL content assist in XHTML: http://docs.jboss.org/tools/whatsnew/jst/jst-news-3.3.0.M3.html
* Navigation from/to bean
* Search usage
* Refactoring:
http://docs.jboss.org/tools/whatsnew/jst/jst-news-3.2.0.M1.html
* New JSF project wizard (JSF 2.2 or older)
* Composite component code assist:
https://issues.jboss.org/browse/JBIDE-4970, http://docs.jboss.org/tools/whatsnew/jst/jst-news-3.2.0.Beta2.html, Validation and refactoring are also available
* EL Validation: http://docs.jboss.org/tools/whatsnew/jst/jst-news-3.2.0.M2.html

OpenShift
---------

* Create a new server adapter from ``OpenShift Explorer''
* More details at http://blog.arungupta.me/getting-started-wildfly-openshift-jboss-developer-studio/

Forge
-----

* Switch to JBoss perspective
* Go to ``Forge Console'', click on play button to start it
* `project-new --named sample`
* `javaee-setup --javaEEVersion 7`
* `jpa-setup --jpaVersion 2.1`
* Install plugin: `addon-install-from-git --url https://github.com/forge/addon-batch`
** Create new Job XML: `batch-new-jobxml --jobXML myJob.xml --reader org.svcc.MyReader --writer org.svcc.MyWriter`
* More details about Batch and Forge at: http://blog.arungupta.me/javaee7-batch-addon-jboss-forge-part1/
* More details about other technologies at: http://blog.arungupta.me/rapid-javaee-development-forge2/

Continuous Delivery
-------------------

