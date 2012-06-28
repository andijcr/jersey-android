jersey-android
==============

A porting of Jersey 1.12 for Android OS.

Essentially jersey-android is the jersey-core,jersey-server,jersey-servlet packed in one Android Library project with some packages and classes removed to make it work on Android.

I removed wadl and osgi packages and support to JSP and EJB.
This was done just to reach a basic working version. They might actually work on Android provided the right javax dependencies are added to jersey-android project.

Read the following paragraphs to use jersey-android.

*CREATE ANDROID PROJECT AND JERSEY-ANDROID AS ANDROID LIBRARY

Create an Android project and include jersey-android as Androdid Library (from Eclipse, right click on project -> Properties -> select Android tab on the left -> Click Add -> Select jersey-android project from your workspace).

*LAUNCH JETTY SERVER FROM AN ACTIVITY

Here it comes some code to launch a Jetty Server and add a Jersey Servlet to it.
Read the following blog post to run Jetty on Android:
http://puregeekjoy.blogspot.com.es/2011/06/running-embedded-jetty-in-android-app.html

```import org.eclipse.jetty.server.Server;
import org.eclipse.jetty.servlet.ServletContextHandler;
import org.eclipse.jetty.servlet.ServletHolder;

import android.app.Activity;
import android.os.Bundle;
import android.util.Log;

import com.sun.jersey.spi.container.servlet.ServletContainer;

public class StartServerActivity extends Activity {

	private Server webServer;

	private final static String LOG_TAG = "Jetty";

	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);

		setContentView(R.layout.main);

		System.setProperty("java.net.preferIPv4Stack", "true");
		System.setProperty("java.net.preferIPv6Addresses", "false");

		webServer = new Server(8080);

		ServletHolder servletHolder = new ServletHolder(ServletContainer.class);
		//DO NOT USE CAUSE IT WON'T WORK ON ANDROID servletHolder.setInitParameter("com.sun.jersey.config.property.packages", "com.famenu.server.resources");
		
		servletHolder.setInitParameter("com.sun.jersey.config.property.resourceConfigClass", "com.sun.jersey.api.core.ClassNamesResourceConfig");
		servletHolder.setInitParameter("com.sun.jersey.config.property.classnames", "com.famenu.server.resources.HelloResource");
		
		ServletContextHandler servletContextHandler = new ServletContextHandler(webServer, "/api", true, false);
		servletContextHandler.addServlet(servletHolder, "/hello");
	
		webServer.setHandler(servletContextHandler);
		

		try {
			webServer.start();
			Log.d(LOG_TAG, "started Web server");

		}
		catch (Exception e) {
			Log.d(LOG_TAG, "unexpected exception starting Web server: " + e);
		}

	}
}
```

```import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;

@Path("/")
public class HelloResource {
	
	@GET
	@Produces("text/plain")
	public String getMsg() {
		
		return "Hello Resource";
		
	}
	
}
```

*AVOID SERVLET initParameter com.sun.jersey.config.property.packages

On Android the Jersey Servlet initParameter com.sun.jersey.config.property.packages doesn't work as described here: http://stackoverflow.com/questions/11210734/jersey-on-jetty-on-android-throws-containerexception-the-resourceconfig-instanc/11211101#11211101

*IGNORE DX CORE-LIBRARY WARNING

jersey-android depends on javax libraries which are not available for Android. In the future I will repackage this libraries into a non-javax package. For now you need to disable --core-library check in dx and compile the .apk with ant. The following steps explains how:

a) Open $ANDROID_HOME/platform-tools/dx
b) Change exec java $javaOpts -jar "$jarpath" "$@" to exec java $javaOpts -jar "$jarpath" --core-library "$@"

(as described here: http://stackoverflow.com/a/11090774/373542 

*BUILD APK AND DEPLOY WITH ANT

Now it's finally to build the project with ant and deploy to your device/emulator. Please note that in step E you have to copy the services folder into the META-INF folder of your apk. You can find service folder within the jersey-android project at libs/META-INF/services. This is necessary for Jersey library otherwise the exception described here will be throwed: http://stackoverflow.com/questions/11191346/jersey-on-jetty-on-android-throws-containerexception-no-webapplication-provide/11194871#11194871


    a) open command line
    b) cd YOUR_PROJECT_FOLDER
    c) generate build.xml by executing command: android update project --path .
    d) build apk by executing command: ant debug
    e) Open your PROJECT_NAME-debug.apk . Navigate to META-INF. Paste services folder that you find within the github project.
    f) Deploy apk to device by executing command: adb install -r "bin/PROJECT_NAME-debug.apk"

*THANKS

Thanks to Martin and Pavel Bucek for their huge help on stackoverflow questions:

http://stackoverflow.com/questions/11210734/jersey-on-jetty-on-android-throws-containerexception-the-resourceconfig-instanc/11211101#11211101

http://stackoverflow.com/questions/11191346/jersey-on-jetty-on-android-throws-containerexception-no-webapplication-provide/11194871#11194871

Thanks to MrTidy OTR for his blog post about Jetty on Android:

http://puregeekjoy.blogspot.com.es/2011/06/running-embedded-jetty-in-android-app.html
