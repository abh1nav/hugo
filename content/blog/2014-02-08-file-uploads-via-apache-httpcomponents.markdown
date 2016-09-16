+++
title = "File uploads in Scala via Apache HttpComponents"
date = "2014-02-08 20:10:05 -0500"
tags = ["http", "apache", "scala", "programming"]
type = "post"
aliases = [
  "/blog/2014/02/08/file-uploads-via-apache-httpcomponents/"
]
+++
  
In order to upload photos to Facebook from the server-side, the Graph API requires you to make a POST request with the image attached as multipart/form-data. Since I couldn't find any native Scala libraries that make this task easy or intuitive, I went with the battle-tested Apache HttpComponents library.
  
Since the app was going to upload photos to Facebook in an on-demand fashion, I went with an HTTPClient that could be reused and was able handle multiple requests concurrently.
  
```scala
val httpClient = {
  val connManager = new PoolingHttpClientConnectionManager()
  HttpClients.custom().setConnectionManager(connManager).build()
}
``` 
  
The next step was to create an entity that contains all the POST data:

```scala
val accessToken = "MY_FACEBOOK_ACCESS_TOKEN"
val file = "/absolute/path/to/file.jpg"
val reqEntity = MultipartEntityBuilder.create()
reqEntity.addPart("source", new FileBody(file))
reqEntity.addPart("access_token", new StringBody(accessToken, ContentType.TEXT_PLAIN))
```
  
In this case `source` is the name of the form field whose value is the image file. The next step is to create a POST request and execute it using our `httpClient`.

```scala
val uri = new URI("https://graph.facebook.com/me/photos")

val httpPost = new HttpPost(uri)
httpPost.setEntity(reqEntity.build())
val response = httpClient.execute(httpPost, HttpClientContext.create())
```
  
Use `EntityUtils` to read the response text:
  
```scala
val entity = response.getEntity
val result = EntityUtils.toString(entity)
if(response != null) response.close()
```
  
Remember to close the response after you're done reading from it. Here's a full example including all the imports:
  
```scala
import java.io.File
import java.net.URI
import org.apache.http.client.methods.{HttpPost, CloseableHttpResponse}
import org.apache.http.client.protocol.HttpClientContext
import org.apache.http.entity.mime.MultipartEntityBuilder
import org.apache.http.entity.mime.content.{StringBody, FileBody}
import org.apache.http.entity.ContentType
import org.apache.http.util.EntityUtils
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager
import org.apache.http.impl.client.HttpClients
import scala.util.Try

object Uploader {

  lazy val httpClient = {
    val connManager = new PoolingHttpClientConnectionManager()
    HttpClients.custom().setConnectionManager(connManager).build()
  }

  def uploadToFacebook(file: File, accessToken: String): Try[String] = {
    val uri = new URI("https://graph.facebook.com/me/photos")

    Try({
      // Create the entity
      val reqEntity = MultipartEntityBuilder.create()

      // Attach the file
      reqEntity.addPart("source", new FileBody(file))

      // Attach the access token as plain text
      val tokenBody = new StringBody(accessToken, ContentType.TEXT_PLAIN)
      reqEntity.addPart("access_token", tokenBody)

      // Create POST request
      val httpPost = new HttpPost(uri)
      httpPost.setEntity(reqEntity.build())

      // Execute the request in a new HttpContext
      val ctx = HttpClientContext.create()
      val response: CloseableHttpResponse = httpClient.execute(httpPost, ctx)

      // Read the response
      val entity = response.getEntity
      val result = EntityUtils.toString(entity)

      // Close the response
      if(response != null) response.close()
      
      result
    })
  }
}
```