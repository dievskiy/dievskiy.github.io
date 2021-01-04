In this mini-blog, I want to describe how I developed [this](https://play.google.com/store/apps/details?id=app.rootstock) app and what solutions and components I used.

The idea of the app is simple: take notes in the messenger-like friendly environment with customizable content structure. Other note-taking apps allow usually only 1 or 2 levels of content abstractions, while this app doesn't have any limit. You can create "workspaces" (which is analog of folder in OS), where you can create "channels" (files) to take notes in. From a technical perspective, it is basically an n-ary tree.

![structure](https://imgur.com/57BJm4h.png)

By the way, I also developed a backend for this app. It runs on python using the FastAPI web framework and is deployed to GCP. There is also simple flask-based [website](https://github.com/dievskiy/fraktus-website).

### Design

For my developer arsenal, I generally picked Android Architecture Components just to practice modern Android dev techniques and simplify some cumbersome problems.

In the app, there are 2 main activities: WorkspaceActivity and ChannelActivity. The former is a nice representation of Workspace entity and the latter - actual place to take notes.

Workspace activity contains Workspace Fragment, which contains "child" workspaces and channels. To navigate through that kind of hierarchical data (from one workspace fragment to another) I used Navigation Component because it simplifies argument passing and is a friend with ViewModel.
Workspace Fragment | Channels Fragment
-------------------------|-------------------------
![](https://imgur.com/SWMcrZm.png)   |  ![](https://imgur.com/XKPZChv.png)

Channel activity represents messages regarding a specific channel. The interesting caveat here is endless scrolling. The problem is quite common for messengers: list of messages is "endless" and storing them all simultaneously is memory-consuming. It is usually solved with custom recyclerview implementation that allows "paginate" data. Luckily, there is also a special Component for that - Paging. It "helps to load and display small chunks of data at a time" and it's data source is compatible with kotlin new feature Flow. With that, I only needed to implement PagingDataAdapter, data mediator to have endless scrolling. It also handles network-local db communication (but in other cases, I ended up in implementing cache-first by myself).

One funny problem I had with design was truncating unicode characters in case of text overflow. The idea was to allow users to write emojis in channels' and workspaces' names to make it more appealing, but android doesn't provide solution out of the box. I also found that this problem [has been present](https://www.notion.so/!%5B%3Chttps://github.com/signalapp/Signal-Android/issues/3266%3E%5D(%3Chttps://github.com/signalapp/Signal-Android/issues/3266%3E)) for a long time. The solution was to use "marquee" ellipsize technique.

### Network

One of the most interesting tasks was to configure proper network communication. In backend, I implemented user authentication with oAuth standards and JWT tokens. But in android, token handling turned out to be quite a challenge.

![](https://articles.cidaas.de/assets/refresh-token-compressor.png)

As you can see, there are few situations that should be considered: access token expiring, refresh token expiring, requests from user while updating tokens and revoking tokens.

The solution can be quite elegant - with help of okHttp interceptors and special Authenticator class that catches 401 (Unauthorized) errors. 401 leads to refreshing token, and if refresh token is also expired, redirects user to login activity.

### Data structure

On the backend side, I ended up using postgres database for storing user data (although postgres might be overkill for that simple app, I wanted still use it as the main database to learn more about its functionality; and if the app needs to be scaled, it will be even more relevant). In postgres, there is very suitable for my goals [ltree type](https://www.notion.so/!%5B%3Chttps://www.postgresql.org/docs/9.1/ltree.html%3E%5D(%3Chttps://www.postgresql.org/docs/9.1/ltree.html%3E)), which represents "labels of data stored in a hierarchical tree-like structure". But on android, in SQLite, there is no such type and hierarchical data should be represented, for example, using additional table with parent-child relationships between entities.

### TODOS

As for now, the biggest problem of the app is absence of messenger-suitable protocols. This leads to the impossibility to implement good real-time usage of the app on multiple devices. In the next versions, the app should use socket connection (also to allow possible "shared" between multiple users channels).

The second feature that will dramatically improve UX is the ability to send media files. I decided not to include this functionality in the first beta version, mostly because of difficulties of implementation on backend, but it definitely should be present in the future.

### Conclusion

Overall, AA Components provide really handy tools for quick app development and testing. In my opinion, there is no much frontend work in android world in our days. Things get pretty straightforward when you have this arsenal of components, which lets a developer to focus on business logic or (which is on the other side of android dev spectrum) specific view behavior. The idea of the project was not to create one more boring note-taking app, but to understand backend-mobile interaction more deeply. 
