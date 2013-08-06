# Roar Engine

Welcome to Roar, an asynchronous, scriptable, server-side **game mechanics engine** that exposes an HTTP Web API for you to consume from your application.

Download a Roar license here: **[http://roarengine.com](http://roarengine.com)**


## Quicklinks

Get started with **Roar documentation available** under `/docs` or at:
**[http://github.com/roarengine/roar-docs/](http://github.com/roarengine/roar-docs/)**

- **[Installation](http://github.com/roarengine/roar-docs/blob/master/docs/setup.md) (start here)**
- [Running the server](http://github.com/roarengine/roar-docs/blob/master/docs/running.md) - start and stop
- [Configuration files](http://github.com/roarengine/roar-docs/tree/master/docs/configuration_files/)
- [Scripting](http://github.com/roarengine/roar-docs/blob/master/docs/concepts/scripting.md)
- [Logging](http://github.com/roarengine/roar-docs/blob/master/docs/concepts/logging.md)

Web API related reference links:

- Web API reference docs: [http://roarengine.com/webapi](http://roarengine.com/webapi)
- Basic API console: [http://roarengine.com/codeconsole](http://roarengine.com/codeconsole/)

Our open source wrappers for the API:

- Unity3D/C#: [http://github.com/roarengine/sdk-unity/](http://github.com/roarengine/sdk-unity/)
- Javascript: [http://github.com/roarengine/sdk-javascript/](http://github.com/roarengine/sdk-javascript/)

## License
Your use of Roar Engine is bound by the [License Terms](http://roarengine.com/license.html).

## Application Stack
The Roar Application Stack comprises the following distinct layers:

- **Web**
  _Apache+PHP web layer_
  Handles incoming HTTP requests and routes to the App layer.

- **Leaderboard**
  _PHP+MySQL leaderboard module_
  A sub-module dedicated to handling leaderboards.

- **Application**
  _C++ Roar socket application_
  The core game engine (the server itself).

- **DB**
  _Postgres app DB layer_
  Used to persist application data (eg. players, items, etc).


## Setting up
Roar is officially supported on **[Ubuntu Server 12.04 64bit](http://www.ubuntu.com/download/server)** [[1]](#footnote-1). We recommend running Roar using **Amazon Web Services** ([AWS](http://aws.amazon.com/)), and as such make references to AWS in our docs from time to time. You may of course run your servers wherever you wish.

Click here for **[detailed installation instructions](http://github.com/roarengine/roar-docs/blob/master/docs/setup.md)**.

We recommend running Roar in AWS with the following starting config:

- Elastic Loadbalancer
- 1x EC2 medium node (web layer)
- 1x EC2 x-large node (application + db + leaderboard)


---

<a id="footnote-1" name="footnote-1"></a>[1] Many developers have successfully reported running Roar on everything from Mac OSX to Solaris. Your mileage _will_ vary. Roar is officially supported **only** on Ubuntu Server 12.04 64bit.
