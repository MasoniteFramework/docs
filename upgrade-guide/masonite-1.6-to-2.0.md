# Masonite 1.6 to 2.0

Masonite 2 brings an incredible new release to the Masonite family. This release brings a lot of new features to Masonite to include new status codes, database seeding, built in cron scheduling, controller constructor resolving, auto-reloading server, a few new internal ways that Masonite handles things, speed improvements to some code elements and so much more. We think developers will be extremely happy with this release.

Upgrading from Masonite 1.6 to Masonite 2.0 shouldn't take very long. On an average sized project, this upgrade should take around 30 minutes. We'll walk you through the changes you have to make to your current project and explain the reasoning behind it.

## Application Configuration 

Masonite 2 adds some improvements with imports. Previously we had to import providers and drivers like:

```text
from masonite.providers.UploadProvider import UploadProvider
```

Because of this, all framework service providers will need to change as well to:

```text
 PROVIDERS = [ # Framework Providers 'masonite.providers.AppProvider.AppProvider', 'app.providers.SessionProvider.SessionProvider', 'masonite.providers.RouteProvider.RouteProvider', # 'entry.providers.ApiProvider.ApiProvider', 'masonite.providers.RedirectionProvider.RedirectionProvider', 'masonite.providers.StartResponseProvider.StartResponseProvider', 'masonite.providers.SassProvider.SassProvider', 'masonite.providers.WhitenoiseProvider.WhitenoiseProvider',
```





