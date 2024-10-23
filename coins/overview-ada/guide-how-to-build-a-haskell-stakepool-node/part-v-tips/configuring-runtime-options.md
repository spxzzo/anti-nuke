# Configuring Glasgow Haskell Compiler Runtime System Options

Depending on hardware specifications, in some implementations of Cardano Node configuring runtime system (RTS) options available for the Glasgow Haskell Compiler (GHC) may improve performance and/or stability.

The RTS offers many options for controlling behaviour. For example, depending on your system specifications and the Cardano Node version, your block-producing node sometimes may miss slot leader checks, as reported using the [gLiveView](../part-iii-operation/starting-the-nodes.md#gliveview) dashboard. Using custom RTS options, you may seek to reduce significantly the number of slot leader checks that your block producer misses, for example due to time spent in garbage collection (GC).

{% hint style="info" %}
For suggestions on reducing the number of missed slot leader checks, see the topic [Reducing Missed Slot Leader Checks](./reducing-missed-slot-leader-checks.md).
{% endhint %}

**To display the default RTS options for Cardano Node:**

* In a terminal window, type:

```bash
cardano-node +RTS --info
```

In the results, the value of the `Flag -with-rtsopts` key displays the default RTS options. For example:

```bash
-T -I0 -A16m -N2 --disable-delayed-os-memory-return
```

For details on available RTS options and setting RTS options using the command line, in the [GHC User's Guide](https://downloads.haskell.org/ghc/8.10.7/docs/users_guide.pdf) see the sections _RTS Options for SMP Parallelism_ on page 136 and _Running a Compiled Program_ on page 172.

For example, to produce runtime system statistics for each garbage collection in addition to using default RTS options, include the following RTS option in your `cardano-node run` command where `<FileName>` is the name of the file in the current folder where you want to output statistics and `<Options>` is the list of options that you use when running your node:

`/usr/local/bin/cardano-node run +RTS -S<FileName> -RTS <Options>`

If you are not satisfied with the performance or stability of an instance of Cardano Node, then subjectively you may notice improvement when adding custom RTS options such as:

```bash
+RTS -N -qg -qb -RTS
```

```bash
+RTS -I0.3 -Iw600 -F1.5 -H2500M -RTS
```

```bash
+RTS -M12g -N -RTS
```

Using similar syntax, you can configure custom RTS options when running the Cardano Command Line Interface (CLI).

**To display the default RTS options for Cardano CLI:**

* In a terminal window, type:

```bash
cardano-cli +RTS --info
```

If you identify different RTS options that noticably improve the performance or stability of your Cardano Node instance, then consider contributing the RTS options to [Coin Cashew](https://www.coincashew.com/) so that other stake pools may also benefit.
