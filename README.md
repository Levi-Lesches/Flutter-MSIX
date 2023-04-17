# Flutter-MSIX
Builds a Windows App Installer (.msix) for your Windows Flutter app, using https://pub.dev/packages/msix.

To use this action, add your Windows Certificate file, in base64 format, to a repository secret and pass that to this workflow:

```yaml
name: Build Windows app MSIX

# Builds a Windows App Installer (.msix) for the Dashboard
# 
# To use this action, add your Windows Certificate file, in base64 format, to a
# repository secret called WINDOWS_CERTIFICATE. This action then: 
# - Installs Flutter and clones your repository
# - Decodes your text certificate into a binary .pfx file
# - Runs flutter pub run msix:create to build and sign your Flutter app
# - Creates a new release and uploads the generated .msix file to it

on:
  push:
    tags: "*"

jobs:
 build:
  runs-on: windows-latest
  env: 
    windows_certificate: ${{ secrets.WINDOWS_CERTIFICATE }}
  steps:
    - name: Generate MSIX
      uses: Levi-Lesches/Flutter-MSIX@v1.0
      with: 
        base64-certificate: ${{ env.windows_certificate }}
        certificate-path: windows_certificate.pfx
        msix-path: build/windows/runner/release/MyApp.msix        
```

Make sure your action runs on `windows-latest` in order to compile to Windows.

- Full instructions on building your own `.msix` files can be found on [Flutter's website](https://docs.flutter.dev/deployment/windows#packaging-and-deployment)
- This package uses [`package:msix`](https://pub.dev/packages/msix) to do the heavy-lifting -- **make sure to follow the setup there first!**
- This package assumes you want to sign your package with a certificate, which is highly recommended. To create a certificate, follow [these](https://www.baeldung.com/openssl-self-signed-cert) instructions to generate your own self-signed certificate with OpenSSL, being sure to end up with a `.pfx` file
- **DO NOT** push your certificate file to your repository, as that defeats the purpose of the certificate! Instead, 
	- add it to your `.gitignore`
	- convert your `.pfx` file to a base64-encoded format using `openssl base64 -in your_certificate.pfx -out base64.txt`
	- Add the entire contents of the `base64.txt` file to a [repository secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository)
	- Use that secret as the value for this workflow's `base64-certificate` parameter
