# Mi3vb18
## Voorbeeld om met Volley in Android Studio data op te vragen

Volley is de bibliotheek die op dit moment (november 2017) door
Google wordt aangeraden om data op te vragen.

Dit voorbeeld bouwt verder op het voorbeeld dat in de vorige les
gebruikt werd (https://github.com/ophalvens/MI3-CordovaVoorbeeld/tree/Stap-5-DATA).

In dit voorbeeld werken we wel met Kotlin in plaats van met
Java. (code: https://github.com/ophalvens/Mi3vb18/blob/master/app/src/main/java/net/ophalvens/mi3vb18/MainActivity.kt).

Voor wie liever de java-syntax leest, kan de code in het zip-bestand 'MainActivity.zip' interessant zijn. 

### Stap -1
Dit project is gebaseerd op een nieuwe app, gebaseerd op de  NavigationDrawerActivity setup wizard.

### Stap 0
* Documentatie over Volley : https://developer.android.com/training/volley/index.html.
* Documentatie over Kotlin : http://kotlinlang.org/docs/tutorials/kotlin-android.html.

### Stap 1 : app.gradle
In je app.gradle :
```
dependencies {
  ...
  compile 'com.android.volley:volley:1.1.1'
}
```
Afhankelijk van de Gradle versie waarmee je werkt, kan je die compile vervangen door *implementation*.
Met *implementation* zullen de volgende builds van je app sneller zijn.

Je kan de releases van Volley bekijken op https://github.com/google/volley/releases.

### Stap 2 : AndroidManifest.xml
Doordat je internet nodig hebt, moet je aan je manifest ook de *android.permission.INTERNET* toevoegen.
```
<manifest ...>
  ... 
  <uses-permission android:name="android.permission.INTERNET" />
  ...
</manifest>
```

### Stap 3 : RequestQueue en url 
In de onCreate fun(ctie) maak je onder de super.onCreate() de volgende
variabelen:

```
override fun onCreate(savedInstanceState: Bundle?) {
  ... 
  val queue = newRequestQueue(this);
  val url = "http://ophalvens.net/mi3/testdb.php";
}
```
 * *val* gebruik je voor variabelen die immutable zijn
 * *val* gebruik je voor variabelen die kunnen veranderen

### Stap 4 : JSONObject aanmaken
Om gegevens mee te kunnen sturen in de POST request voor dit voorbeeld, maken we eerst een JSONObject aan.
```
...
val jObj = JSONObject()
try {
    //jObj.put("id",32); // indien je een specifieke ID wilt opvragen
    jObj.put("bewerking", "get")
    jObj.put("table", "producten")
} catch (e: JSONException) {
    e.printStackTrace()
}
```
### Stap 5 : Maak de request aan
```
...
val jsObjRequest = JsonObjectRequest(Request.Method.POST, url, jObj,
    Response.Listener<JSONObject> { response ->
        tv_main.text = ""
        val ts = StringBuilder("")
        var tArray: JSONArray? = null
        var tObjectRecord: JSONObject? = null

        try {
            tArray = response.getJSONArray("data")
            for (i in 0 until tArray!!.length()) {
                tObjectRecord = tArray.getJSONObject(i)
                ts.append("id:" + tObjectRecord!!.getInt("PR_ID") + " - ")
                ts.append("product:" + tObjectRecord.getString("PR_naam") + " - ")
                ts.append("prijs:" + tObjectRecord.getString("prijs"))
                ts.append("\n")
            }
            tv_main.text = ts.toString()

        } catch (e: JSONException) {
            e.printStackTrace()
        }
    }, Response.ErrorListener { error -> tv_main.text = error.cause.toString() })
```
* *tv_main* Is de TextView waarin we de herwerkte response in plaatsen.
* Aangezien we hier een String gaan concateneren werken we met een *StringBuilder* omdat dit meer efficiënt is qua geheugengebruik.
* De response zelf is een **JSONObject**.
* In de response zit in "data" een **JSONArray** waarin we alle nuttige waarden hebben gestopt op de server.
* In deze JSONArray zitten **JSONObject**en.
* In deze JSONObjecten zitten waarden met namen zoals *PR_ID*, *PR_naam*, *prijs* (zie voorbeeld Cordova).

De parameters voor deze JsonObjectRequest zijn :
* de **method**
* de **uri**
* de **data** die je meestuurt in je request
* een **Response.Listener** on success (Response.Listener< JSONObject> omdat we een JSONObject verwachten van de server )
* een **Response.ErrorListener**

### Stap 6 : het requestObject aan de queue toevoegen
Om de request effectief uit te kunnen voeren, wil je deze ook aan de queue toevoegen: 
```
...
queue.add(jsObjRequest)
```
### Stap 7 : de API aanpassen
Wat je verstuurt met Volley komt niet in de $_POST of $_GET variabelen van de PHP pagina te staan.
Om dat op te lossen, kijken we naar de body van de request in de PHP pagina :
```
...
$body = file_get_contents('php://input');
$postvars = json_decode($body, true);
$id = $postvars["id"];
$table = $postvars["table"];
$bewerking = $postvars["bewerking"];
...
```
Je kan dit in meer detail bekijken in het voorbeeld van de API op https://github.com/ophalvens/mi3Cordova18/blob/Stap5/php/testdb.php .
