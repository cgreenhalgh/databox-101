# Databox/ OneDrive notes

Thinking about sharing data from another app (e.g. phone) via files in OneDrive.

Note, there are 
[some differences](https://docs.microsoft.com/en-us/onedrive/developer/rest-api/getting-started/release-notes?view=odsp-graph-online#view-deltas)
between the operation of the API for OneDrive personal vs OneDrive for Business and SharePoint.

Some notes...

See
[Accessing shared items](https://docs.microsoft.com/en-gb/onedrive/developer/rest-api/api/shares_get?view=odsp-graph-online)
```
string sharingUrl = "https://onedrive.live.com/redir?resid=1231244193912!12&authKey=1201919!12921!1";
string base64Value = System.Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(sharingUrl));
string encodedUrl = "u!" + base64Value.TrimEnd('=').Replace('/','_').Replace('+','-');
```

For a folder, 
`GET /shares/{shareIdOrUrl}/driveItem?$expand=children`

Note, "For OneDrive for Business and SharePoint, the Shares API always requires authentication and cannot be used to access anonymously shared content without a user context."

See 
[app authentication with Microsoft Graph](https://developer.microsoft.com/en-us/graph/docs/authorization/auth_overview)
using OAuth2.0 & Open ID Connect.
Requires Azure AD v2.0, set up via 
[Azure portal app registrations](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)
(was 
[Microsoft Application Registration portal](https://apps.dev.microsoft.com/))

See
[node.js REST sample](https://github.com/microsoftgraph/nodejs-connect-rest-sample)
(Android & iOS use SDK).

Oooh... Oh :-(
[Track changes](https://docs.microsoft.com/en-us/onedrive/developer/rest-api/api/driveitem_delta?view=odsp-graph-online)
but "OneDrive for Business and SharePoint
The delta action is only available on the root item of a drive e.g. /drive/root."
([release notes](https://docs.microsoft.com/en-us/onedrive/developer/rest-api/getting-started/release-notes?view=odsp-graph-online#view-deltas)) :-(

"File facet
OneDrive for Business and SharePoint: The file facet does not include MIME type, SHA1 or CRC32 hash values.
OneDrive personal: The file facet does not include the QuickXOr hash value."

Also has 
[webhooks](https://docs.microsoft.com/en-us/onedrive/developer/rest-api/concepts/using-webhooks?view=odsp-graph-online),
but only one callback URL per application.

Note that onedrive files are versioned.
Single item upload is up to 4MB; larger files are uploaded through a resumable session (in 320kB - 10MB chunks).
But there is no concatenation to existing files.

Has file checkout/checkin for lock-based concurrent editing.

Note also, "To download files from OneDrive in a JavaScript app you cannot use the /content API, since this responds with a 302 redirect. ...
Instead, your app needs to select the @microsoft.graph.downloadUrl property, which returns the same URL that /content would have redirected to. This URL can then be requested directly using XMLHttpRequest. Because these URLs are pre-authenticated they can be retrieved without a CORS preflight request."
i.e. `GET /drive/items/{item-id}?select=id,@microsoft.graph.downloadUrl`.
See
[working with cors](https://docs.microsoft.com/en-us/onedrive/developer/rest-api/concepts/working-with-cors?view=odsp-graph-online)