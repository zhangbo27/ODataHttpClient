[![CircleCI](https://circleci.com/gh/iwate/ODataHttpClient/tree/master.svg?style=svg)](https://circleci.com/gh/iwate/ODataHttpClient/tree/master)
[![NuGet version](https://badge.fury.io/nu/ODataHttpClient.svg)](https://badge.fury.io/nu/ODataHttpClient)

<p>The simplest implementation from of OData client.</p>

<h2 id="install">Install</h2>

<pre><code>$ dotnet add package ODataHttpClient
PS&gt; Install-Package  ODataHttpClient
</code></pre>

<h2 id="restrictions">Restrictions</h2>

<ul>
<li>Not support XML (JSON only)</li>

<li>Not support Query Builder</li>
</ul>

<h2 id="usage">Usage</h2>

<h3 id="simplerequest">Simple Request</h3>

<pre><code>var client = new HttpClient();
var odata = new ODataClient(client);
var reqest = Request.Get($"{endpoint}/Products?$inlinecount=allpages");
var response = await odata.SendAsync(request);

if (response.Success) 
{
    var total = response.ReadAs&lt;int&gt;("$['odata.count']");
    var products = response.ReadAs&lt;IEnumerable&lt;Product&gt;&gt;("$.value");
}
</code></pre>

<h3 id="batchrequest">Batch Request</h3>

<pre><code>var client = new HttpClient();
var odata = new ODataClient(client);
var batch = new BatchRequest($"{endpoint}/$batch")
{
    Requests = new []
    {
        Request.Post($"{endpoint}/Products", new 
        { 
            Name = "Sample"
        }),
        Request.Post($"{endpoint}/$1/$links/Categories", new 
        {
            url = $"{endpoint}/Categories(0)"
        })
    }
};

var responses = await odata.BatchAsync(batch);

if (responses.All(res =&gt; res.Success)) 
{
    var id = response.First().ReadAs&lt;int&gt;("$.Id");
}
</code></pre>

<h3 id="setodataelementtype">Set OData Element Type</h3>

<p>In default, OData element type is not contains in payload. If you want add element type, you sould use <code>type</code> parameter of Request factory method.</p>

<pre><code>Request.Post("...", payload, type: "ODataDemo.Product")
</code></pre>

<h3 id="changeodataelementtypekey">Change OData Element Type Key</h3>

<p>In default, OData element type key is <code>odata.type</code>. If you want change key to other, you should use <code>typeKey</code> parameter of Request factory method.</p>

<pre><code>Request.Post("...", payload, type: "ODataDemo.Product", typeKey: "@odata.type")
</code></pre>

<h2 id="credential">Credential</h2>

<h3 id="basicauth">Basic Auth</h3>

<pre><code>var odata = new ODataClient(client, "username", "password");
</code></pre>

<h3 id="custom">Custom</h3>

<p>You should modify default settings of client which be passed to ODataClient constructor or implemet <code>ICredentialBuilder</code></p>

<pre><code>public class CustomCredentialBuilder : ODataHttpClient.Credentials.ICredentialBuilder
{
    public void Build(HttpClient client, HttpRequestMessage message)
    {
        // implement custom credential logic.
    }
}
</code></pre>

<p>And pass the builder instance to 2nd parameter constructor.</p>

<pre><code>var odata = new ODataClient(client, new CustomCredentialBuilder());
</code></pre>

<h2 id="jsonserializersettings">Json Serializer Settings</h2>

<p>In default, ODataHttpClient use <a href="http://www.odata.org/documentation/odata-version-2-0/json-format/#PrimitiveTypes">OData V2 JSON Serialization Format</a>. <br />
If you change general json format, can select a way of three.</p>

<h3 id="1globallevelsettings">1. Global level settings</h3>

<pre><code>ODataHttpClient.Serializers.JsonSerializer.DefaultJsonSerializerSettings 
            = ODataHttpClient.Serializers.JsonSerializer.GeneralJsonSerializerSettings; // new Newtonsoft.Json.JsonSerializerSettings();
</code></pre>

<h3 id="2instancelevelsettings">2. Instance level settings</h3>

<pre><code>var odata = new ODataHttpClient(httpClient, ODataHttpClient.Serializers.JsonSerializer.GeneralJsonSerializerSettings);
var request = odata.RequestFacotry.Get("..."); // MUST use RequestFactory
</code></pre>

<h3 id="3requestlevelsettings">3. Request level settings</h3>

<pre><code>var settings = ODataHttpClient.Serializers.JsonSerializer.GeneralJsonSerializerSettings;
var request = Request.Get("...", settings); // pass settings
var response = await odata.SendAsync(request);
var data = response.ReadAs&lt;dynamic&gt;(settings); // pass settings
</code></pre>
