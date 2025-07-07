# Shizuku

## Contexto

Ao desenvolver aplicativos que requerem root, o método mais comum é executar alguns comandos no shell su. Por exemplo, existe um aplicativo que usa o comando `pm enable/disable` para ativar/desativar componentes.

Este método tem grandes desvantagens:

1. **Extremamente lento** (Criação de múltiplos processos)
2. Necessita processar textos (**Super não confiável**)
3. A possibilidade é limitada aos comandos disponíveis
4. Mesmo que o ADB tenha permissões suficientes, o aplicativo requer privilégios root para executar

Shizuku usa uma maneira completamente diferente. Veja a descrição detalhada abaixo.

## Guia do usuário & Download

<https://shizuku.rikka.app/>

## Como o Shizuku funciona?

Primeiro, precisamos falar sobre como os aplicativos usam as APIs do sistema. Por exemplo, se o aplicativo quer obter os apps instalados, todos sabemos que devemos usar `PackageManager#getInstalledPackages()`. Isso é, na verdade, uma comunicação entre processos (IPC).

Android usa `binder` para fazer este tipo de IPC. O `binder` permite que o lado do servidor saiba o uid e pid do lado do cliente, para que o servidor do sistema possa verificar se o aplicativo tem a permissão para fazer a operação.

Normalmente, se existe um "gerenciador" (ex: `PackageManager`) para os apps usarem, deve haver um "serviço" (ex: `PackageManagerService`) no processo do servidor do sistema. Podemos pensar simplesmente que se o app quiser usar APIs do sistema, ele precisa se comunicar com o processo do servidor do sistema.

O Shizuku guia os usuários para executar um processo, o servidor Shizuku, com root ou ADB primeiro. Quando o aplicativo inicia, o `binder` para o servidor Shizuku também será enviado para o aplicativo.

A característica mais importante que o Shizuku fornece é algo como ser um intermediário para receber solicitações do aplicativo, enviá-las para o servidor do sistema e enviar de volta os resultados. Você pode ver a função `transactRemote` para mais detalhes.

Assim, alcançamos nosso objetivo de usar APIs do sistema com permissão mais alta. E para o aplicativo, é quase idêntico ao uso direto das APIs do sistema.

## Guia do desenvolvedor

### API & exemplo

https://github.com/RikkaApps/Shizuku-API

### Migrando da versão pré-v11

> Os aplicativos existentes ainda funcionam, é claro.

https://github.com/RikkaApps/Shizuku-API#migration-guide-for-existing-applications-use-shizuku-pre-v11

### Atenção

1. Permissões ADB são limitadas

   O ADB tem permissões limitadas e diferentes em várias versões do sistema. Você pode ver as permissões concedidas ao ADB [aqui](https://github.com/aosp-mirror/platform_frameworks_base/blob/master/packages/Shell/AndroidManifest.xml).

   Antes de chamar a API, você pode usar `ShizukuService#getUid` para verificar se o Shizuku está executando o ADB do usuário, ou usar `ShizukuService#checkPermission` para verificar se o servidor tem permissões suficientes.

2. Limitação de API oculta a partir do Android 9

   A partir do Android 9, o uso das APIs ocultas é limitado para aplicativos normais. Por favor, use outros métodos (como <https://github.com/LSPosed/AndroidHiddenApiBypass>).

3. Android 8.0 & ADB

   Atualmente, a forma como o serviço Shizuku obtém o processo do aplicativo é combinar `IActivityManager#registerProcessObserver` e `IActivityManager#registerUidObserver` (26+) para garantir que o processo do aplicativo seja obtido.

4. O uso direto de `transactRemote` requer atenção

   * A API pode ser diferente em diferentes versões do Android, por favor, verifique cuidadosamente. Além disso, o `android.app.IActivityManager` tem a forma aidl na API 26 e posterior.
   * `SystemServiceHelper.getTransactionCode` pode não obter o código de transação correto.

## Desenvolvendo o próprio Shizuku

### Compilação

- Clone com `git clone --recurse-submodules`
- Execute a tarefa gradle `:manager:assembleDebug` ou `:manager:assembleRelease`

A tarefa `:manager:assembleDebug` gera um servidor depurável. Você pode anexar um depurador ao `shizuku_server` para depurar o servidor.

## Licença

Todos os arquivos de código neste projeto estão licenciados sob Apache 2.0

Sob a seção 6 do Apache 2.0, especificamente:

* Você está **PROIBIDO** de usar os arquivos de imagem `manager/src/main/res/mipmap*/ic_launcher*.png`, exceto para exibir o próprio Shizuku.

* Você está **PROIBIDO** de usar `Shizuku` como nome do aplicativo ou usar `moe.shizuku.privileged.api` como ID do aplicativo ou declarar permissão `moe.shizuku.manager.permission.*`.