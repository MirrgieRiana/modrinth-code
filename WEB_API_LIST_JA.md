# Modrinth Web API 一覧 (v2)

このドキュメントは `apps/docs/public/openapi.yaml` を基に自動生成した概要です。ベースURLは `https://api.modrinth.com/v2` です。各エンドポイントのメソッド、主要パラメーターを列挙します。

## `/search`

- **GET**: Search projects
  - パラメーター:
    - `query` (query, 任意) — The query to search for
    - `facets` (query, 任意) — Facets are an essential concept for understanding how to filter out results.  These are the most commonly used facet types: - `project_type` - `categories` (loaders are lumped in with categories in search) - `versions` - `client_side` - `server_side` - `open_source`  Several others are also available for use, though these should not be used outside very specific use cases. - `title` - `author` - `follows` - `project_id` - `license` - `downloads` - `color` - `created_timestamp` (uses Unix timestamp) - `modified_timestamp` (uses Unix timestamp) - `date_created` (uses ISO-8601 timestamp) - `date_modified` (uses ISO-8601 timestamp)  In order to then use these facets, you need a value to filter by, as well as an operation to perform on this value. The most common operation is `:` (same as `=`), though you can also use `!=`, `>=`, `>`, `<=`, and `<`. Join together the type, operation, and value, and you've got your string. ``` {type} {operation} {value} ```  Examples: ``` categories = adventure versions != 1.20.1 downloads <= 100 ```  You then join these strings together in arrays to signal `AND` and `OR` operators.  ##### OR All elements in a single array are considered to be joined by OR statements. For example, the search `[["versions:1.16.5", "versions:1.17.1"]]` translates to `Projects that support 1.16.5 OR 1.17.1`.  ##### AND Separate arrays are considered to be joined by AND statements. For example, the search `[["versions:1.16.5"], ["project_type:modpack"]]` translates to `Projects that support 1.16.5 AND are modpacks`. 
    - `index` (query, 任意) — The sorting method used for sorting search results
    - `offset` (query, 任意) — The offset into the search. Skips this number of results
    - `limit` (query, 任意) — The number of results returned by the search


## `/project/{id|slug}`

- **GET**: Get a project
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project

- **PATCH**: Modify a project
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project
  - リクエストボディあり

- **DELETE**: Delete a project
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project


## `/projects`

- **GET**: Get multiple projects
  - パラメーター:
    - `ids` (query, 必須) — The IDs and/or slugs of the projects

- **PATCH**: Bulk-edit multiple projects
  - パラメーター:
    - `ids` (query, 必須) — The IDs and/or slugs of the projects
  - リクエストボディあり


## `/projects_random`

- **GET**: Get a list of random projects
  - パラメーター:
    - `count` (query, 必須) — The number of random projects to return


## `/project`

- **POST**: Create a project
  - リクエストボディあり


## `/project/{id|slug}/icon`

- **PATCH**: Change project's icon
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project
    - `ext` (query, 必須) — Image extension
  - リクエストボディあり

- **DELETE**: Delete project's icon
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project


## `/project/{id|slug}/check`

- **GET**: Check project slug/ID validity
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project


## `/project/{id|slug}/gallery`

- **POST**: Add a gallery image
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project
    - `ext` (query, 必須) — Image extension
    - `featured` (query, 必須) — Whether an image is featured
    - `title` (query, 任意) — Title of the image
    - `description` (query, 任意) — Description of the image
    - `ordering` (query, 任意) — Ordering of the image
  - リクエストボディあり

- **PATCH**: Modify a gallery image
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project
    - `url` (query, 必須) — URL link of the image to modify
    - `featured` (query, 任意) — Whether the image is featured
    - `title` (query, 任意) — New title of the image
    - `description` (query, 任意) — New description of the image
    - `ordering` (query, 任意) — New ordering of the image

- **DELETE**: Delete a gallery image
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project
    - `url` (query, 必須) — URL link of the image to delete


## `/project/{id|slug}/dependencies`

- **GET**: Get all of a project's dependencies
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project


## `/project/{id|slug}/follow`

- **POST**: Follow a project
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project

- **DELETE**: Unfollow a project
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project


## `/project/{id|slug}/schedule`

- **POST**: Schedule a project
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project
  - リクエストボディあり


## `/project/{id|slug}/version`

- **GET**: List project's versions
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project
    - `loaders` (query, 任意) — The types of loaders to filter for
    - `game_versions` (query, 任意) — The game versions to filter for
    - `featured` (query, 任意) — Allows to filter for featured or non-featured versions only


## `/version/{id}`

- **GET**: Get a version
  - パラメーター:
    - `id` (path, 必須) — The ID of the version

- **PATCH**: Modify a version
  - パラメーター:
    - `id` (path, 必須) — The ID of the version
  - リクエストボディあり

- **DELETE**: Delete a version
  - パラメーター:
    - `id` (path, 必須) — The ID of the version


## `/project/{id|slug}/version/{id|number}`

- **GET**: Get a version given a version number or ID
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project
    - `id|number` (path, 必須) — The version ID or version number


## `/version`

- **POST**: Create a version
  - リクエストボディあり


## `/version/{id}/schedule`

- **POST**: Schedule a version
  - パラメーター:
    - `id` (path, 必須) — The ID of the version
  - リクエストボディあり


## `/versions`

- **GET**: Get multiple versions
  - パラメーター:
    - `ids` (query, 必須) — The IDs of the versions


## `/version/{id}/file`

- **POST**: Add files to version
  - パラメーター:
    - `id` (path, 必須) — The ID of the version
  - リクエストボディあり


## `/version_file/{hash}`

- **GET**: Get version from hash
  - パラメーター:
    - `hash` (path, 必須) — The hash of the file, considering its byte content, and encoded in hexadecimal
    - `algorithm` (query, 必須) — The algorithm of the hash
    - `multiple` (query, 任意) — Whether to return multiple results when looking for this hash

- **DELETE**: Delete a file from its hash
  - パラメーター:
    - `hash` (path, 必須) — The hash of the file, considering its byte content, and encoded in hexadecimal
    - `algorithm` (query, 必須) — The algorithm of the hash
    - `version_id` (query, 任意) — Version ID to delete the version from, if multiple files of the same hash exist


## `/version_file/{hash}/update`

- **POST**: Latest version of a project from a hash, loader(s), and game version(s)
  - パラメーター:
    - `hash` (path, 必須) — The hash of the file, considering its byte content, and encoded in hexadecimal
    - `algorithm` (query, 必須) — The algorithm of the hash
  - リクエストボディあり


## `/version_files`

- **POST**: Get versions from hashes
  - リクエストボディあり


## `/version_files/update`

- **POST**: Latest versions of multiple project from hashes, loader(s), and game version(s)
  - リクエストボディあり


## `/user/{id|username}`

- **GET**: Get a user
  - パラメーター:
    - `id|username` (path, 必須) — The ID or username of the user

- **PATCH**: Modify a user
  - パラメーター:
    - `id|username` (path, 必須) — The ID or username of the user
  - リクエストボディあり


## `/user`

- **GET**: Get user from authorization header


## `/users`

- **GET**: Get multiple users
  - パラメーター:
    - `ids` (query, 必須) — The IDs of the users


## `/user/{id|username}/icon`

- **PATCH**: Change user's avatar
  - パラメーター:
    - `id|username` (path, 必須) — The ID or username of the user
  - リクエストボディあり

- **DELETE**: Remove user's avatar
  - パラメーター:
    - `id|username` (path, 必須) — The ID or username of the user


## `/user/{id|username}/projects`

- **GET**: Get user's projects
  - パラメーター:
    - `id|username` (path, 必須) — The ID or username of the user


## `/user/{id|username}/follows`

- **GET**: Get user's followed projects
  - パラメーター:
    - `id|username` (path, 必須) — The ID or username of the user


## `/user/{id|username}/payouts`

- **GET**: Get user's payout history
  - パラメーター:
    - `id|username` (path, 必須) — The ID or username of the user

- **POST**: Withdraw payout balance to PayPal or Venmo
  - パラメーター:
    - `id|username` (path, 必須) — The ID or username of the user
    - `amount` (query, 必須) — Amount to withdraw


## `/user/{id|username}/notifications`

- **GET**: Get user's notifications
  - パラメーター:
    - `id|username` (path, 必須) — The ID or username of the user


## `/notification/{id}`

- **GET**: Get notification from ID
  - パラメーター:
    - `id` (path, 必須) — The ID of the notification

- **PATCH**: Mark notification as read
  - パラメーター:
    - `id` (path, 必須) — The ID of the notification

- **DELETE**: Delete notification
  - パラメーター:
    - `id` (path, 必須) — The ID of the notification


## `/notifications`

- **GET**: Get multiple notifications
  - パラメーター:
    - `ids` (query, 必須) — The IDs of the notifications

- **PATCH**: Mark multiple notifications as read
  - パラメーター:
    - `ids` (query, 必須) — The IDs of the notifications

- **DELETE**: Delete multiple notifications
  - パラメーター:
    - `ids` (query, 必須) — The IDs of the notifications


## `/report`

- **POST**: Report a project, user, or version
  - リクエストボディあり

- **GET**: Get your open reports
  - パラメーター:
    - `count` (query, 任意) — 


## `/report/{id}`

- **GET**: Get report from ID
  - パラメーター:
    - `id` (path, 必須) — The ID of the report

- **PATCH**: Modify a report
  - パラメーター:
    - `id` (path, 必須) — The ID of the report
  - リクエストボディあり


## `/reports`

- **GET**: Get multiple reports
  - パラメーター:
    - `ids` (query, 必須) — The IDs of the reports


## `/thread/{id}`

- **GET**: Get a thread
  - パラメーター:
    - `id` (path, 必須) — The ID of the thread

- **POST**: Send a text message to a thread
  - パラメーター:
    - `id` (path, 必須) — The ID of the thread
  - リクエストボディあり


## `/threads`

- **GET**: Get multiple threads
  - パラメーター:
    - `ids` (query, 必須) — The IDs of the threads


## `/message/{id}`

- **DELETE**: Delete a thread message
  - パラメーター:
    - `id` (path, 必須) — The ID of the message


## `/project/{id|slug}/members`

- **GET**: Get a project's team members
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project


## `/team/{id}/members`

- **GET**: Get a team's members
  - パラメーター:
    - `id` (path, 必須) — The ID of the team

- **POST**: Add a user to a team
  - パラメーター:
    - `id` (path, 必須) — The ID of the team
  - リクエストボディあり


## `/teams`

- **GET**: Get the members of multiple teams
  - パラメーター:
    - `ids` (query, 必須) — The IDs of the teams


## `/team/{id}/join`

- **POST**: Join a team
  - パラメーター:
    - `id` (path, 必須) — The ID of the team


## `/team/{id}/members/{id|username}`

- **PATCH**: Modify a team member's information
  - パラメーター:
    - `id` (path, 必須) — The ID of the team
    - `id|username` (path, 必須) — The ID or username of the user
  - リクエストボディあり

- **DELETE**: Remove a member from a team
  - パラメーター:
    - `id` (path, 必須) — The ID of the team
    - `id|username` (path, 必須) — The ID or username of the user


## `/team/{id}/owner`

- **PATCH**: Transfer team's ownership to another user
  - パラメーター:
    - `id` (path, 必須) — The ID of the team
  - リクエストボディあり


## `/tag/category`

- **GET**: Get a list of categories


## `/tag/loader`

- **GET**: Get a list of loaders


## `/tag/game_version`

- **GET**: Get a list of game versions


## `/tag/license`

- **GET**: Get a list of licenses


## `/tag/license/{id}`

- **GET**: Get the text and title of a license
  - パラメーター:
    - `id` (path, 必須) — The license ID to get the text of


## `/tag/donation_platform`

- **GET**: Get a list of donation platforms


## `/tag/report_type`

- **GET**: Get a list of report types


## `/tag/project_type`

- **GET**: Get a list of project types


## `/tag/side_type`

- **GET**: Get a list of side types


## `/updates/{id|slug}/forge_updates.json`

- **GET**: Forge Updates JSON file
  - パラメーター:
    - `id|slug` (path, 必須) — The ID or slug of the project
    - `neoforge` (query, 任意) — Whether to include NeoForge versions. Can be `only` (NeoForge-only versions), `include` (both Forge and NeoForge versions), or omitted (Forge-only versions).


## `/statistics`

- **GET**: Various statistics about this Modrinth instance

