# [sig-rpc.github.io](sig-rpc.github.io)

The Reasonable Performance Computing SIG's Website & Knowledge-base, where user-submitted guides can help those writing software to identify and address performance problems in their code.

The goal of [the website](sig-rpc.github.io) is to make profiling and optimisation guidance accessible to even novice programmers. The main pre-requisite for readers is that they're able to read through code and recognise how it will execute.

### Contributing

Regardless of your familiarity with code performance and optimisation, we welcome your contribution. Contributions may be as small as providing feedback on how a guide is unclear or could be improved, or as large as a fully drafted new guide ready for publication.

Authors with highly technical backgrounds can easily make assumptions, which aren't clear to those less experienced. This is why even feedback from those new to software performance is important!

Ideas and feedback can be submitted via the repository's [issue templates](https://github.com/sig-rpc/sig-rpc.github.io/issues/new/choose).

New articles can be submitted directly as pull requests. It's worth using an existing guide as a template, these can be found in `_profilers/` and `_optimisations/` respectively.

For the full contribution guidance, including a loose style guide, please refer to [`CONTRIBUTING.md`](.github/CONTRIBUTING.md).

*Something is better than nothing.*

### Building Locally

The website is a static website built with the Ruby package [Jekyll](jekyllrb).

It's easiest to run Jekyll under a Unix environment, Windows users are encouraged to use the Windows Subsystem for Linux (WSL).

If you don't already have Ruby installed, please refer to Ruby's [installation guide](https://www.ruby-lang.org/en/documentation/installation/).

With Ruby setup, the below command executed in the root of the repository will install the dependencies including Jekyll.

```sh
bundle install
```

Following this, you can either run the website as a development server or build a static copy.

To execute a development server, that will rebuild files on modification.

```sh
bundle exec jekyll serve
```

To build a static copy of the website.

```sh
bundle exec jekyll build
```

### Acknowledgements

The website is only made possible through user contributions, in most cases authors have been identified on the individual guides they contributed to. For a complete picture check out GitHub's [contributors interface](https://github.com/sig-rpc/sig-rpc.github.io/graphs/contributors).

The SIG-RPC community has been supported by both the [Society of Research Software Engineering](https://society-rse.org/) and the [Software Sustainability Institute](https://www.software.ac.uk/).