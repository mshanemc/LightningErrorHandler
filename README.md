# Lightning Error Handler

This is just the error handler bits from /mshanemc/ltngbase.  I'm trying to unbundle that now that sfdx and it's oss plugin make it easy to add repos from github.

displays errors from apex callbacks or force:recordData.  **I prefer option 1 whenever possible.**




## option 1 (method)  --**Preferred**

If each component that wants to throw an error wants to host its own error handler, you can use a simpler version.
``` html
<c:LightningErrorHandler aura:id="leh"/>
```

example of apex callback (where `a` is the callback response)

``` javascript
if (state === "SUCCESS") {
    //do your happy path logic
}  else if (state === "ERROR") {
    component.find('leh').passErrors(a.getError());
```

The error handler will only respond to method calls from its parent component.

example using Lightning Data Service/force:recordData in EDIT mode where the aura:id is "frd"

``` javascript
component.find("frd").saveRecord(
    $A.getCallback(function(saveResult){
        if (saveResult.state === "SUCCESS"){
            //happy path logic here
        } else if (saveResult.state === "ERROR"){
            component.find("leh").passErrors(saveResult.error);
        }
    })
)
```

## option 2 (event)

``` html
<aura:registerEvent name="handleCallbackError" type="c:handleCallbackError"/>
<c:LightningErrorHandler />
```

Simply pass in the errors object from `.getErrors` to a handleCallbackError event and it'll handle the rest.  You can optionally name the ErrorHandler and put that name in the handleCallbackError event

Install like this:
example of callback (where `a` is the callback response)

``` javascript
if (state === "SUCCESS") {
    //do your happy path logic
}  else if (state === "ERROR") {
    var appEvent = $A.get("e.c:handleCallbackError");
    appEvent.setParams({
        "errors" : a.getError()
    });
    appEvent.fire();
```

If you have multiple of these on the page, you can also scope errors to a specific instance of LightningErrorHanlder like this:

``` html

<aura:registerEvent name="handleCallbackError" type="c:handleCallbackError"/>
<c:LightningErrorHandler errorHandlerName="specificName"/>

```
Then pass that LEH's name when you fire the error event:

``` javascript
if (state === "SUCCESS") {
    //do your happy path logic
}  else if (state === "ERROR") {
    var appEvent = $A.get("e.c:handleCallbackError");
    appEvent.setParams({
        "errors" : a.getError(),
        "errorComponentName" : "specificName"
    });
    appEvent.fire();
```



## Limitations

* I haven't built duplicate handling yet.
* I haven't done any sort of event bubbling management (stopPropogation, etc) if you're using option 1, so be cautious about hanging one of these on every branch of your component tree and firing off events.  :)
* I recommend using option1 only where you know there's only one of these on the page, or you very carefully apply the ErrorHandlerName

<a href="https://githubsfdeploy.herokuapp.com?owner=mshanemc&repo=LightningErrorHandler">
  <img alt="Deploy to Salesforce"
       src="https://raw.githubusercontent.com/afawcett/githubsfdeploy/master/deploy.png"/>
</a>

[![Deploy](https://deploy-to-sfdx.com/dist/assets/images/DeployToSFDX.svg)](https://deploy-to-sfdx.com/deploy?template=https://github.com/mshanemc/lightningErrorHandler)
