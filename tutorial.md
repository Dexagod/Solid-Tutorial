## Getting started {#intro}
In this tutorial, we make use of a local installation of the [Community Solid Server (CSS)](https://github.com/CommunitySolidServer/CommunitySolidServer) to provide an introduction to Solid. 

## Requirements
You will need the following to follow this tutorial (not mandatory):
* [git](https://git-scm.com/)
* [Node.js](https://nodejs.org/en/)
  * Preferably 16.x but at the time of writing 14.x should also work.
* A text editor

### Getting started
While you can install the server as a global application,
we work directly from the source in this tutorial.
You do this by cloning the server from the git repository and installing it:

```shell
git clone https://github.com/CommunitySolidServer/CommunitySolidServer.git
cd CommunitySolidServer
npm install
```

After that you start the server by running the following command from the same folder:
```shell
npm start
```


This starts your own Solid server, which you then see at http://localhost:3000/.

### Example Solid HTTP requests
All interactions with the server happen through HTTP requests,
with each HTTP method having its own use.
Below are some common examples.
You run these using `curl` on the command line,
or by manually constructing the queries in an HTTP client such as [Hoppscotch](https://hoppscotch.io/).

#### GETting resources
GET is used to request resources:

```shell
curl http://localhost:3000/
```
```turtle
@prefix dc: <http://purl.org/dc/terms/>.
@prefix ldp: <http://www.w3.org/ns/ldp#>.
@prefix posix: <http://www.w3.org/ns/posix/stat#>.
@prefix xsd: <http://www.w3.org/2001/XMLSchema#>.

<> a <http://www.w3.org/ns/pim/space#Storage>, ldp:Container, ldp:BasicContainer, ldp:Resource;
    dc:modified "2022-01-11T12:00:06.000Z"^^xsd:dateTime;
    ldp:contains <index.html>.
```

Note that this is a turtle representation of the root container.
CSS supports content negotiation for many formats,
in case n-triples are preferred these can be requested by sending the corresponding Accept header for example.

```shell
curl -H "Accept: application/n-triples"  http://localhost:3000/
```
```
<http://localhost:3000/> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/ns/pim/space#Storage> .
<http://localhost:3000/> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/ns/ldp#Container> .
<http://localhost:3000/> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/ns/ldp#BasicContainer> .
<http://localhost:3000/> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/ns/ldp#Resource> .
<http://localhost:3000/> <http://purl.org/dc/terms/modified> "2022-01-11T12:00:06.000Z"^^<http://www.w3.org/2001/XMLSchema#dateTime> .
<http://localhost:3000/> <http://www.w3.org/ns/ldp#contains> <http://localhost:3000/index.html> .
```

And in case the HTML representation is needed, as seen in the previous step,
you can send a `text/html` Accept header along.

#### POSTing new resources
Post is used to create new resources.
We use the `-v` flag in the examples here to see the response headers,
since the Location header returns the URL of this new resource.

```shell
curl -v -X POST -H "Content-Type: text/plain" -d "abc" http://localhost:3000/
```

```
Location: http://localhost:3000/823df3de-5766-4d2a-97d7-2a79a7093c57
```

The Slug header is used to recommend a name for the resource to the server:

```shell
curl -v -X POST -H "Slug: custom" -H "Content-Type: text/plain" -d "abc" http://localhost:3000/
```
```
Location: http://localhost:3000/custom
```

You can see this resource at http://localhost:3000/custom.
Note that the Slug header is a suggestion,
servers are not required to use it.

#### PUTting new data
PUT is used to write data to a specific location.
In case that location already exists, the data there is overwritten.
If not, a new resource is created.

```shell
curl -X PUT -H "Content-Type: text/turtle" -d "<ex:s> <ex:p> <ex:o>." http://localhost:3000/a/b/c
```
Doing a GET to http://localhost:3000/a/b/c shows that the resource was created.
Note that this single request also created the intermediate containers 
http://localhost:3000/a/ and http://localhost:3000/a/b/, which you can also access now.

As mentioned, you can also overwrite the data at a location:
```shell
curl -X PUT -H "Content-Type: text/turtle" -d "<ex:s2> <ex:p2> <ex:o2>." http://localhost:3000/a/b/c
```
This command completely replaces the data found at http://localhost:3000/a/b/c.

#### PATCHing resources
PATCH is used to update resources.
This is useful if you do not want to completely overwrite a resource with a PUT
and only want to make a smaller change.

All Solid servers are required to support [N3 Patch](https://solid.github.io/specification/protocol#n3-patch)
to modify RDF resources. 
The following request modifies the resource we created in the previous step.
```shell
curl -X PATCH -H "Content-Type: text/n3" --data-raw "@prefix solid: <http://www.w3.org/ns/solid/terms#>. _:rename a solid:InsertDeletePatch; solid:inserts { <ex:s3> <ex:p3> <ex:o3>. }." http://localhost:3000/a/b/c
```

The above command will append the triple `<ex:s3> <ex:p3> <ex:o3>.` to the RDF resource,
which can be verified by GETting http://localhost:3000/a/b/c.

#### DELETE a resource
Finally, DELETE is used to remove resources.

```shell
curl -X DELETE http://localhost:3000/a/b/c
```

This completely removes the resource from the server.
The parent containers will remain,
so http://localhost:3000/a/ and http://localhost:3000/a/b/ still exist.


### Editing PERMISSIONS (WAC)
Solid makes use of the Web Access Controls (WAC) authorization system to determine what interactions an agent is allowed to perform on the resources in your pod. 
This system uses Access Control List (ACL) resources that define who can perform what operations on a resource.
In general, these resources have an `.acl` extension, but you can always find the corresponding ACL in the response header when GETting a resource. The ACL resource that determines access to the root container is http://localhost:3000/.acl.

```turtle
@prefix acl: <http://www.w3.org/ns/auth/acl#>.
@prefix foaf: <http://xmlns.com/foaf/0.1/>.

# Give all agents Read, Write, and Control permissions on everything
<#authorization>
    a               acl:Authorization;
    acl:agentClass  foaf:Agent;
    acl:mode        acl:Read, acl:Write, acl:Append, acl:Control;
    acl:accessTo    <./>;
    acl:default     <./>.
```

A full explanation of all triples can be found in the [WAC specification](https://solid.github.io/web-access-control-spec/). 
This acl document defines that the current container - the root container in this case (`acl:accessTo <./>`) is provides access for reading resources, writing resources, appending to resources and controlling resources (`acl:mode acl:Read, acl:Write, acl:Append, acl:Control;`) by all agents (`acl:agentClass foaf:Agent`).

This is the reason why, until now, we were able to easily add, update and delete resources on the created pod. As the Solid server is currently running in a local environment, the risks of allowing everyone to interact with your pod are minimal. In a Web environment however, this is of course something that we cannot allow.

### Setting up a test pod on the Web
To setup a test pod on the web, we will make use of a Solid server hosted by us on the Web. These servers are reset every night, so if you could not finish the full exercise in time, do not worry if you cannot find your pod the next day.

For the exercises part, please register your own pod at [https://pod.playground.solidlab.be/](https://pod.playground.solidlab.be/idp/register/)

For those that are interested in setting up their own pod in a more permanent pod hosting space, feel free to look at [this overview of Solid Pod Providers](https://solidproject.org/users/get-a-pod#get-a-pod-from-a-pod-provider) that are currently available!


### Updating resources on the Web
With this new pod created, we can navigate to the root of our pod, located at https://pod.playground.solidlab.be/pod_name/, where `pod_name` is the name you entered in the form when registering a new pod.

Here we see the following overview:
```
Contents of ruben
pod.playground.solidlab.be > pod_name

    profile/
    README
```

However, if we click on the `profile/` container, we get an error screen telling us we cannot access this resource, even though we are working on our own pod. This is because you are currently not authenticated, so the server has no way of verifying who you are.


To update resources in an authenticated setting, you can use any online editor that supports Solid authentication. For this exercises session, we make use of a very simple Web editor: https://solideditor.patrickhochstenbach.net/

At the top left here, you see a login button. If we want to see authenticated content, we need to login first. 
As the `Solid IDP` here, you can enter the following: https://pod.playground.solidlab.be/. This is the URL of your Solid Identity Provider. This can be found on your WebID as the following triple:
`<#me> solid:oidcIssuer <https://pod.playground.solidlab.be/> . `

Upon clicking `consent` to indicate that you are ok with this Web editor application interacting with resources on your pod, the application will resume control and you will be logged in. You can tell by the cat icon at the top left of your editor.

To edit resources, we first need to tell the application which resource we would like to edit. This value should be entered in the `Resource URL:` field.

If we enter the private profile container that we could not access via the Web interface (do not forget to substitute pod_name for the name of your pod!): 
`https://pod.playground.solidlab.be/pod_name/profile/` 
we see that now we can view the resource, as we are now authenticated ourselves and can prove to the Solid server that we have the right to view the resource.
This resource however cannot be edited, as it is a container.

#### Editing the pod root acl
Now that we have our editor, we can look into why we only have limited access to our pod. If we navigate to the root ACL file of our pod in the editor (again do not forget to substitute the pod_name in the url):
https://pod.playground.solidlab.be/pod_name/.acl
We see that where the owner of the pod has full control, the public permissions now only allow the reading of resources in this single container.

```turtle
# Root ACL resource for the agent account
@prefix acl: <http://www.w3.org/ns/auth/acl#>.
@prefix foaf: <http://xmlns.com/foaf/0.1/>.

# The homepage is readable by the public
<#public>
    a acl:Authorization;
    acl:agentClass foaf:Agent;
    acl:accessTo <./>;
    acl:mode acl:Read.

# The owner has full access to every resource in their pod.
# Other agents have no access rights,
# unless specifically authorized in other .acl resources.
<#owner>
    a acl:Authorization;
    acl:agent <https://pod.playground.solidlab.be/ruben/profile/card#me>;
    ...
    # All resources will inherit this authorization, by default
    acl:default <./>;
    # The owner has all of the access modes allowed
    acl:mode
        acl:Read, acl:Write, acl:Control.
```

If we want to set all resources on our pod publicly readable by default, unless specified otherwise by a resource ACL, we can update this root .acl resource to make the public resource access the default for the whole pod. While this is not something you typically would do, this might make the process of working with the pod for the first time a bit easier.

By including the `acl:default <./>`, we indicate to our server that public read access should be the default in all resources contained by this container, unless specified otherwise down the line.

```turtle
# Root ACL resource for the agent account
@prefix acl: <http://www.w3.org/ns/auth/acl#>.
@prefix foaf: <http://xmlns.com/foaf/0.1/>.

# The homepage is readable by the public
<#public>
    a acl:Authorization;
    acl:agentClass foaf:Agent;
    acl:accessTo <./>;
    # All resources will inherit this authorization, by default
    acl:default <./>;
    acl:mode acl:Read.

```




### Notifications in Solid
As Solid enables the construction of decentralized networks of data pods, we need some way to communicate information between these pods.
For this, the [Linked Data Notifications](https://www.w3.org/TR/ldn/) specification was created.

If a WebID advertises an inbox link, agents on the Web can send messages to this pod by making a HTTP POST request to this inbox URI.

To send a request to a friend about 
```turtle
"@prefix as: <https://www.w3.org/ns/activitystreams>.

<> a as:Note.
  as:summary: "I will be on vacation from 14-11 until 17-11."
```

We can now save this note as `note.ttl`, and send it to the inbox of a collegue located at https://pod.playground.solidlab.be/colleague_podo not hesitate to contact us,d/inbox/ with the following command:

```
curl -X POST -H "Content-Type: text/turtle" --data-binary "@note.ttl" https://pod.playground.solidlab.be/colleague_pod/inbox/
```

The notification will not be readable for you anymore, as you cannot view your colleague's inbox, but your colleague can now process this notification and wish you a good vacation in return!



### More information

This wraps up the short tutorial on Solid.
If you want more information on Solid or the Community Solid Server, please look around on the 