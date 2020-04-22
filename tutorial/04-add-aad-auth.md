<!-- markdownlint-disable MD002 MD041 -->

在本练习中，你将扩展上一练习中的应用程序，以支持 Azure AD 的身份验证。 若要获取所需的 OAuth 访问令牌以调用 Microsoft Graph，这是必需的。 在此步骤中，将[.net 的 Microsoft 身份验证库（MSAL）](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet)集成到应用程序中。

1. 通过在包含**GraphTutorial**的目录中打开 CLI 并运行以下命令来初始化[.net 开发密钥存储](/aspnet/core/security/app-secrets)。

    ```Shell
    dotnet user-secrets init
    ```

1. 使用以下命令将应用程序 ID 和所需范围的列表添加到机密存储区。 将`YOUR_APP_ID_HERE`替换为你在 Azure 门户中创建的应用程序 ID。

    ```Shell
    dotnet user-secrets set appId "YOUR_APP_ID_HERE"
    dotnet user-secrets set scopes "User.Read;Calendars.Read"
    ```

## <a name="implement-sign-in"></a>实施登录

在本节中，您将创建可与 Graph SDK 一起使用的身份验证提供程序，还可用于使用[设备代码流](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-device-code)显式请求访问令牌。

### <a name="create-an-authentication-provider"></a>创建身份验证提供程序

1. 在名为**Authentication**的**GraphTutorial**目录中创建一个新目录。
1. 在名为 " **DeviceCodeAuthProvider.cs** " 的**身份验证**目录中创建一个新文件，并将以下代码添加到该文件中。

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/DeviceCodeAuthProvider.cs" id="AuthProviderSnippet":::

请考虑此代码执行的操作。

- 它使用 MSAL `IPublicClientApplication`实现来请求和管理令牌。
- `GetAccessToken`函数：
  - 登录用户（如果他们尚未使用设备代码流登录）。
  - 确保返回的令牌总是使用`AcquireTokenSilent`函数（如果未过期，则返回缓存的令牌）进行全新，并在令牌过期时刷新令牌。
- 它实现`IAuthenticationProvider`接口，以便 graph SDK 可以使用类对 graph 调用进行身份验证。

## <a name="sign-in-and-display-the-access-token"></a>登录并显示访问令牌

在本节中，您将更新应用程序以调用`GetAccessToken`该函数，这将对用户进行登录。 您还将添加代码以显示令牌。

1. 将以下函数添加到 `Program` 类。

    :::code language="csharp" source="../demo/GraphTutorial/Program.cs" id="LoadAppSettingsSnippet":::

1. 将以下代码添加到紧跟`Main`在`Console.WriteLine(".NET Core Graph Tutorial\n");`该行后面的函数中。

    :::code language="csharp" source="../demo/GraphTutorial/Program.cs" id="InitializationSnippet":::

1. 将以下代码添加到紧跟`Main`在`// Display access token`该行后面的函数中。

    ```csharp
    Console.WriteLine($"Access token: {accessToken}\n");
    ```

1. 生成并运行应用程序。 应用程序将显示 URL 和设备代码。

    ```Shell
    PS C:\Source\GraphTutorial> dotnet run
    .NET Core Graph Tutorial

    To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code F7CG945YZ to authenticate.
    ```

    > [!TIP]
    > 如果遇到错误，请将您的**Program.cs**与[GitHub 上的示例](https://github.com/microsoftgraph/msgraph-training-dotnet-core/blob/master/demo/GraphTutorial/Program.cs)进行比较。

1. 打开浏览器并浏览到显示的 URL。 输入提供的代码并登录。 完成后，返回到应用程序并选择 " **1"。显示**访问令牌选项以显示访问令牌。
