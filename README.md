# Azure Code Signing
With the Azure Code Signing Action for Github, you can sign your files with an Azure Code Signing certificate.

<!-- something about onboarding -->

# Example
```yaml
on:
  push:
    branches: [main]

jobs:
  build-and-sign:
    runs-on: windows-latest
    name: Build app and sign files with Azure Code Signing
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Setup .NET Core SDK
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: 6.0.x

      - name: Install dependencies
        run: dotnet restore App

      - name: Build
        run: dotnet build --configuration Release --no-restore WpfApp

      - name: Sign files with Azure Code Signing
        uses: azure/azure-code-signing-action@v1.0.0
        with:
          azure-tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          azure-client-id: ${{ secrets.AZURE_CLIENT_ID }}
          azure-client-secret: ${{ secrets.AZURE_CLIENT_SECRET }}
          endpoint: https://wus2.codesigning.azure.net/
          code-signing-account-name: vscx-codesigning
          certificate-profile-name: vscx-certificate-profile
          files-folder: ${{ github.workspace }}\App\App\bin\Release\net6.0-windows
          files-folder-filter: exe,dll
          file-digest: SHA256
          timestamp-rfc3161: http://timestamp.acs.microsoft.com
          timestamp-digest: SHA256
```

## Usage

### Authentication
Behind the scenes, the action uses [EnvironmentCredential](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.environmentcredential) as the primary method of authentication to Azure. The same variables are exposed as inputs and then set to action-scoped environment variables.

#### App Registration
```yaml
# The Azure Active Directory tenant (directory) ID.
azure-tenant-id: ${{ secrets.AZURE_TENANT_ID }}

# The client (application) ID of an App Registration in the tenant.
azure-client-id: ${{ secrets.AZURE_CLIENT_ID }}

# A client secret that was generated for the App Registration.
azure-client-secret: ${{ secrets.AZURE_CLIENT_SECRET }}
```

#### Azure Active Directory
```yaml
# The username, also known as upn, of an Azure Active Directory user account.
azure-username: ${{ secrets.AZURE_USERNAME }}

# The password of the Azure Active Directory user account. Note this does not support accounts with MFA enabled.
azure-password: ${{ secrets.AZURE_PASSWORD }}
```

### Account Details
```yaml
# The Code Signing Account endpoint. The URI value must have a URI that aligns to the region your Code Signing Account and Certificate Profile you are specifying were created in during the setup of these resources.
endpoint: https://wus2.codesigning.azure.net/

# The Code Signing Account name.
code-signing-account-name: my-account-name

# The Certificate Profile name.
certificate-profile-name: my-profile-name

# An opaque string value that you can provide to correlate sign requests with your own workflows such as build identifiers or machine names.
correlation-id: 32C50B44-D8D0-457A-B0FD-9D8220DE91C7
```

### File Specification

#### Files Folder
This strategy allows you to specify a folder that contains all the files you want signed. There are options available for narrowing the focus as well. For example, you can use the `files-folder-filter` input to specify that you only want `exe` files to be signed.

```yaml
# The folder containing files to be signed. Can be combined with the file-catalog input.
files-folder: ${{ github.workspace }}\App\App\bin\Release\net6.0-windows

# A comma separated list of file extensions that determines which types of files will be signed in the folder specified by the files-folder input. Any file type not included in this list will not be signed. If this input is not used, all files in the folder will be signed.
files-folder-filter: dll,exe,msix

# A boolean value (true/false) that indicates if the folder specified by the files-folder input should be searched recursively. The default value is false.
files-folder-recursive: true

# An integer value that indicates the depth of the recursive search toggled by the files-folder-recursive input.
files-folder-depth: 2
```

#### Files Catalog
This strategy allows you to specify a precise list of files to be signed.

```yaml
# A file containing a list of relative paths to the files being signed. The paths should be relative to the location of the catalog file. Each file path should be on a separate line. Can be combined with the files-folder input.
files-catalog: ${{ github.workspace }}\catalog.txt
```

### Digest Algorithm
```yaml
# The name of the digest algorithm used for hashing the file being signed. The default value is SHA256.
file-digest: SHA384
```

### Timestamping
```yaml
# A URL to an RFC3161 compliant timestamping service.
timestamp-rfc3161: http://timestamp.acs.microsoft.com

# The name of the digest algorithm used for timestamping.
timestamp-digest: SHA256
```

### Append Signature
```yaml
# A boolean value (true/false) that indicates if the signature should be appended. If no primary signature is present, this signature is made the primary signature instead.
append-signature: true
```

### Description
```yaml
# A description of the signed content.
description: My signed content.

# A Uniform Resource Locator (URL) for the expanded description of the signed content.
description-url: https://github.com/azure/azure-code-signing-action
```

### Digest
```yaml
# Generates the digest to be signed and the unsigned PKCS7 files at this path. The output digest and PKCS7 files will be path\FileName.dig and path\FileName.p7u. To output an additional XML file, see `generate-digest-xml`.
generate-digest-path: ${{ github.workspace }}\digest

# A boolean value (true/false) that indicates if an XML file is produced when the `generate-digest-path` input is used. The output file will be Path\FileName.dig.xml.
generate-digest-xml: true

# Creates the signature by ingesting the signed digest to the unsigned PKCS7 file at this path. The input signed digest and unsigned PKCS7 files should be path\FileName.dig.signed and path\FileName.p7u.
ingest-digest-path: ${{ github.workspace }}\digest

# A boolean value (true/false) that indicates if only the digest should be signed. The input file should be the digest generated by the `generate-digest-path` input. The output file will be File.signed.
sign-digest: true
```

### Pages Hashes
```yaml
# A boolean value (true/false) that indicates if page hashes should be generated for executable files.
generate-page-hashes: true

# A boolean value (true/false) that indicates if page hashes should be suppressed for executable files. The default is determined by the SIGNTOOL_PAGE_HASHES environment variable and by the wintrust.dll version. This input is ignored for non-PE files.
suppress-page-hashes: true
```

### PKCS7
```yaml
# A boolean value (true/false) that indicates if a Public Key Cryptography Standards (PKCS) #7 file is produced for each specified content file. PKCS #7 files are named path\filename.p7.
generate-pkcs7: true

# Options for the signed PKCS #7 content. Set value to "Embedded" to embed the signed content in the PKCS #7 file, or to "DetachedSignedData" to produce the signed data portion of a detached PKCS #7 file. If this input is not used, the signed content is embedded by default.
pkcs7-options: Embedded

# The object identifier (OID) that identifies the signed PKCS #7 content.
pkcs7-oid: 1.3.6.1.5.5.7.3.3
```

### Enhanced Key Usage
```yaml
# The enhanced key usage (EKU) that must be present in the signing certificate. The usage value can be specified by OID or string. The default usage is "Code Signing" (1.3.6.1.5.5.7.3.3).
enhanced-key-usage: 1.3.6.1.5.5.7.3.3
```

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
