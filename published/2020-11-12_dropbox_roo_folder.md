I'm using a business dropbox account and have access to the teams shared folders through the various interfaces.
Why then when using the API does it only show my personal folder when connecting via the dropbox API?

According to the [dropbox documentation](https://www.dropbox.com/lp/developers/reference/dbx-team-files-guide#namespaces)
it is to do with namespaces. It is also possible to override this behavious and set you namespace to be the
root folder, giving you full access via the API that we get from the web interface. It's not that hard,
you just need to set a header and off you go.

Well, almost, first of all the [dropbox_api gem](https://github.com/Jesus/dropbox_api/pull/73) will need to
merge my PR, giving access to the `root_info` - this is needed to determine the root namespace_id.

Then set the additional header via the built-in middleware stack within the gem:

```ruby
client = DropboxApi::Client.new("VofXAX8D...")

namespace_id = client.get_current_account.root_info.root_namespace_id

client.middleware.prepend do |connection|
  connection.headers['Dropbox-API-Path-Root'] = "{\".tag\": \"namespace_id\", \"namespace_id\": \"#{namespace_id}\"}"
end
```

Hopefully this will help someone out there.

> This is dependant on having your `Dropbox App` correctly configured with the necessary permissions.
