# Quick Start
From your terminal of choice:

```
mkdir quick-start
cd quick-start
npm init
npm install -g typescript
tsc --init
mkdir src
npm install --save-dev @types/xrm
npm install --save dataverse-ify@2
```

Now you need to auth against Dataverse using:
```
npx dataverse-auth@2 [environment]
```
E.g.
```
npx dataverse-auth@2 contosoorg.crm.dynamics.com
```

Now you can generate your types
```
npx dataverse-gen@2 init
```

Follow the instructions - you can select the entities/actions/functions to include in the config

Now open VSCode:
```
Code .
```
You'll want to update your tsconfig to be similar to:

```
{
	"compilerOptions": {
		"target": "ES2017",
		"module": "commonjs",
		"moduleResolution": "node",
		"lib": [
			"ES2017",
			"dom"
		],
		"rootDir": "src",
		"strict": true,
		"alwaysStrict": true,
		"strictFunctionTypes": true,
		"strictNullChecks": true,
		"strictPropertyInitialization": true,
		"forceConsistentCasingInFileNames": true,
		"noImplicitAny": true,
		"noImplicitReturns": true,
		"noImplicitThis": true,
		"noFallthroughCasesInSwitch": true,
		"noUnusedLocals": true,
		"noUnusedParameters": true,
		"emitDecoratorMetadata": false,
		"experimentalDecorators": false,
		"downlevelIteration": true,
		"declaration": false,
		"sourceMap": true,
		"pretty": true,
		"esModuleInterop": true
	}
}
```

You can now start to write code using the `XrmContextDataverseClient` by creating a test as follows:
```
import { Entity, setMetadataCache, XrmContextDataverseClient } from "dataverse-ify";
import { SetupGlobalContextIfUndefined } from "dataverse-ify/lib/webapi/node";
import {
  Account,
  AccountAttributes,
  accountMetadata,
  Contact,
  contactMetadata,
  metadataCache,
} from "../../dataverse-gen";

export someFunction() {
  setMetadataCache(metadataCache);
  const serviceClient = new XrmContextDataverseClient(Xrm.WebApi);

  const contact1 = {
    logicalName: contactMetadata.logicalName,
    firstname: "Jazzy",
    lastname: "Jeff",
  } as Contact;

  // Create contact
  contact1.id = await serviceClient.create(contact1);

  // Create Account - notice primarycontactid is a simple EntityReference
  const account1 = {
    logicalName: accountMetadata.logicalName,
    name: "Contoso",
    primarycontactid: Entity.toEntityReference(contact1),
  } as Account;

  account1.id = await serviceClient.create(account1);

  // Retrieve Account (notice enum AccountAttributes)
  const accountRetrieved = await serviceClient.retrieve<Account>(accountMetadata.logicalName, account1.id, [
    AccountAttributes.PrimaryContactId,
  ]);

  console.log(accountRetrieved);
}
```

This code will run inside a model-driven form since the Xrm.* api is available. If you wanted to run it inside an integration test you would use the technique described in [integration-testing](integration-testing.md) .

For an example of a webpack configuration, check out a sample template repo [dataverse-jswebresource-template](https://github.com/scottdurow/dataverse-jswebresource-template)

If you don't want to use the `XrmContextDataverseClient` - that's fine you can just use the [dataverse-ify methods to transform the raw WebApi request/responses](using-with-service-client.md).

