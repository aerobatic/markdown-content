# Traffic Control

On most application platforms there is a single version of the application live
at any given time. Aerobatic works differently in that every version deployed is always accessible. Within the application portal you can adjust what percentage of sessions should be given which version; whether it be 100% of traffic to a single version or split the traffic to two or more versions in whatever proportions you like. This capability can be used to enable both testing of different experiences as well as [green/blue deployments](http://martinfowler.com/bliki/BlueGreenDeployment.html) where new
versions are initially released to a small subset of overall users to reduce
risk with frequent releases.

## A/B Testing Analytics
The `__config__` global object that Aerobatic injects into the `index.html` page contains a `versionName` attribute that you can pass along in your web analytics calls. This allows you to define audience segments based on the app version in order to measure how different versions perform relative to each other. For example with Google Analytics you might define a session level [custom dimension](https://developers.google.com/analytics/devguides/platform/customdimsmets) corresponding to the app version then define reports which segment based on this dimension.

## Force version URL
Every time you deploy an Aerobatic app to the cloud a unique version id is
generated that can be appended to the base app URL to force that specific
version; bypassing the weighted random version selection process.
```
http://<app_name>.aerobaticapp.com?version=<unique_version_id>`
```

The ability to access any version via URL, even if no live traffic is being directed
to it, opens up a number of streamlined workflows:

* Developers can provide QA testers the URL of a new version where it can be
tested fully integrated in production; eliminating potential test/dev
environment discrepancies.
* Allow clients/stakeholders to preview and approve new versions before opening
it up to real customers.
* Send a link to a specific version to select customers without impacting users
who arrive at the app normally.
* Experience the complete chronology of how an application evolved over time.
