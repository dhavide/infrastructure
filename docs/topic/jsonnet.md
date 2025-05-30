# Jsonnet in our Infrastructure

We use [Jsonnet](https://jsonnet.org/) in our infrastructure in a few
places:

1. To generate [eksctl](https://eksctl.io/) config for our AWS clusters
2. To generate config to pass to helm, for deploying both support charts and hubs
3. To handle Grafana Dashboard definitions, both in this repository and [upstream](https://github.com/jupyterhub/grafana-dashboards).

## Why?

In all these cases, Jsonnet allows us to manage repetitive config while
balancing two factors:

1. Manual need for engineers to copy paste config while making some changes each time based on what they are copying and pasting
2. Fully programmatic tweaking of config via Python (via changes to our `deployer` package)

(1) is bad because that's not what our time should be spent on, plus we are (thankfully) human and will make mistakes. (2) is bad because it makes it difficult to know *where* a config is coming from, tempts us into writing super complex config, and can cause issues with our [right to replicate](https://2i2c.org/right-to-replicate/) guarantee (communities may need to take the deployer code, not just the config, when they leave).

Jsonnet offers a nice middle ground because it is *side effect free* - you can generate output JSON, but not much else. So the temptations of doing super complex things in Python is removed. Think of it as a superset of JSON, and you're good to go.

As we try to automate more of our infrastructure, Jsonnet can be a very
powerful tool for us. Be aware of the factors laid out here when introducing it into any particular place in the infrastructure, but don't feel the need to be overly cautious either.

## How do I learn it?

The [Jsonnet Tutorial](https://jsonnet.org/learning/tutorial.html) is a good
place to learn the language. Just the basics should take you a long way. If
you already know JSON and Python (particularly list comprehensions, dict comprehensions and string formatting with `%s`) you should find it familiar.

The short [design](https://jsonnet.org/articles/design.html) page on the Jsonnet
site is also helpful.

## Import path when `deployer` renders `.jsonnet` files

When the `deployer` renders `.jsonnet` files, it puts two directories in
the import path of the command:

1. The directory where the `.jsonnet` file being rendered lives
2. The directory, under `config/clusters` where the cluster containing the support chart or the hub being rendered lives.

This allows us to read other config files (in [YAML](https://jsonnet.org/ref/stdlib.html#std-parseYaml) or [JSON](https://jsonnet.org/ref/stdlib.html#std-parseJson) form) from either
of those directories and avoid repetition.

Each time the deployer renders `.jsonnet`, it will also print the exact command
it is using for this rendering. That should also help you with debugging!