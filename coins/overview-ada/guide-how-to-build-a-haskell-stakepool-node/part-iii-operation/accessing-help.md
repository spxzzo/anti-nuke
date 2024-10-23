# Accessing Built-in Help

The Cardano command line interface `cardano-cli` provides a collection of tools for key generation, transaction construction, certificate creation and other important tasks.

The interface is organized in a hierarchy of commands and subcommands. Each command and subcommand includes built-in documentation describing command syntax as well as options. If you want more information about `cardano-cli` commands and options, then the built-in documentation may help you.

**To access built-in documentation for the Cardano command line interface:**

1. In a terminal window on a computer having `cardano-cli` installed, type the following command to display top-level help including a list of available commands:

```bash
cardano-cli
```

1. To display built-in documentation for a specific command including a list of available subcommands, type the following where `<Command>` is the name of the command for which you want to display help:

```bash
cardano-cli <Command> --help
```

1. To display built-in documentation for a specific subcommand including a list of available options, type the following where `<Command>` and `<Subcommand>` are the names of the command and subcommand, respectively, for which you want to display help:

```bash
cardano-cli <Command> <Subcommand> --help
```

For example, typing `cardano-cli` informs you that a `conway` command is available for performing actions related to the Conway network era. To display additional help for the `cardano-cli conway` command, type:

```bash
cardano-cli conway --help
```

The command-specific help that displays informs you that a `node` command is available for performing actions related to node operation. To display additional help for the `cardano-cli conway node` command, type:

```bash
cardano-cli conway node --help
```

The command-specific help that displays includes the `key-gen-KES` subcommand used to generate a KES key pair. A current KES key pair is required to operate your stake pool. To display the available options for the `cardano-cli conway node key-gen-KES` command, type:

```bash
cardano-cli conway node key-gen-KES --help
```

The built-in documentation is available to help you understand commands that you execute when setting up and operating your stake pool.

{% hint style="info" %}
If you need additional help, the [Cardano Forum](https://forum.cardano.org/) is available as a resource where you may ask questions, share information and learn.
{% endhint %}