# Mariner

![OSS Lifecycle](https://img.shields.io/osslifecycle/indeedeng/Mariner.svg)

## Introduction

A node.js library for analyzing open source library dependencies.

Mariner takes an input list of dependencies, fetches details about them from GitHub,
and outputs a file containing funding information for each project owner, and a list
of issues for each project.

NOTE: This library is in the experimental stage, so expect breaking changes
even if the version number does not indicate that.

### Renaming the default branch from master

We anticipate renaming the default branch of this repository from `master` to `main`.
GitHub is planning to have a smooth easy conversion process/tool for later this year.
When that tool is available, we plan to take full advantage of it.
[Read more about it here](https://github.com/github/renaming/blob/main/README.md).

## Getting Started Using Mariner

If you just want to USE Mariner, you don't need to do a git clone.
Instead, create your own new node project, and install the oss-mariner package via npm:
`npm install oss-mariner`

### Step-by-step

1. Create a new project folder and use `npm init` to make it a node project.
1. Copy the contents of `runFasterCode.ts` into `index.js` in the new project.
   1.1. <https://github.com/indeedeng/Mariner/blob/master/examples/runFasterCode.ts>
1. Comment out the existing line that imports mariner.
1. Uncomment the line saying how mariner would normally be imported.
1. Convert the TypeScript code to JavaScript by
   1.1. Remove the `public` keywords from class members.
   1.1. Remove the `implements Xxxx` from the FancyLogger class declaration.
   1.1. Remove all the type declarations (like `: string`).
1. Replace the path.join lines with simple hard-coded filenames: `exampleData.json` and `output.json`.
1. Create an exampleData.json file or copy it in from Mariner.
1. Run `npm install oss-mariner`
1. Add `"type": "module"` to `package.json`.
1. Run `node index.js`.

### More details (possibly outdated)

Mariner can be called from Javascript or from Typescript. You can see an example here:
<https://github.com/indeedeng/Mariner/blob/master/examples/runOldCode.ts>

Mariner is in transition from the old way of accessing GitHub data (REST) to the new way (GraphQL)

To invoke mariner using the new GraphQL code you can see an example here:
<https://github.com/indeedeng/Mariner/blob/master/examples/runFasterCode.ts>

If you are using mariner with the new GraphQL code, Invoke the finder(), passing the
appropiate parameters in finder.findIssues(),

```
const token = getFromEnvOrThrow('MARINER_GITHUB_TOKEN');  // from an environment variable
const inputFilePath = process.env.MARINER_INPUT_FILE_PATH || path.join(__dirname, '..', '..', 'examples', 'exampleData.json');
const outputFilePath = process.env.MARINER_OUTPUT_FILE_PATH || path.join(__dirname, '..', '..', 'examples', 'output.json');

const finder = new IssueFinder(logger);
finder.findIssues(token, labels, repositoryLookupName)
    .then((issues) => {
        let issueCount = 0;
        issues.forEach((issuesForRepo) => {
            issueCount += issuesForRepo.length;
        });

        convertToRecord(issues);
        logger.info(`Found ${issueCount} issues in ${issues.size} projects\n`);
        logger.info(`Saved issue results to: ${outputFilePath}`);
    })
.catch((err) => {
        logger.error(err.message);
        console.log(err);
    });

```

If you are using the examples/runOldCode.ts file, (using the old REST code that is very slow)
invoke the DependencyDetailsRetriever.run() method, passing appropriate parameters:

```
const ddr = new DependencyDetailsRetriever();
const githubToken = Process.env.GITHUB_TOKEN; // from an environment variable
const inputFilePath = '<full path to your input file>';
const outputFilePath = '<full path to the file that ddr should create>';
const abbreviated = false; // OPTIONAL; default is false; true will exclude some dependencies
ddr.run(githubToken, inputFilePath, outputFilePath, abbreviated);

```

For both the runOldCode.ts and runFasterCode.ts files you must create a token.
The GitHub token must be a valid personal access token. It does not require any permissions beyond
the default, so when you create it you can leave all the boxes unchecked. Be careful not to
share your token with anyone. If it gets exposed, revoke it and create a replacement.
See https://github.com/settings/tokens/new for how to create a token.

The input file is a JSON file in the format:

-   At the top level is a map/object, where each entry consists of a dependency as the key,
    and the number of projects that depend on that library as the value.
-   Each dependency can be identified by a complete URL or just the owner/repo string.
-   Example complete url: "https://api.github.com/repos/spring-projects/spring-framework": 19805,
-   Example owner/repo strings: "square/retrofit": 5023,
-   The project count value is mostly ignored, but is used by the "abbreviated" feature.
-   See examples/exampleData.json for a complete example.

The output file is a JSON file in the format:

-   (We'll add a definition of the format later.
    For now, you can look at examples/output.json after running the app)

We don't recommend using the `abbreviated` feature.
It will omit entries that have fewer than a hard-coded number of projects that depend on them.

## Getting Help

The [Open Source team at Indeed](https://opensource.indeedeng.io/), who can be reached at opensource@indeed.com.

## How To Contribute

Read the Code of Conduct and Contact the Maintainers before making any changes or a PR.
If an issue doesn’t already exist that describes the change you want to make, we recommend
creating one. If an issue does exist, please comment on it saying that you are starting to
work on it, to avoid duplicating effort.

## Getting Started Developing Mariner

Clone the repository from GitHub.

Run `npm ci` to install the libraries used in the project. Read more about [npm ci here.](https://blog.npmjs.org/post/171556855892/introducing-npm-ci-for-faster-more-reliable)

Follow the instructions in examples/runFasterCode.ts or examples/runOldCode.ts to configure the input and output files. NOTE: An example input file is included, in the examples directory.

Run `npm run build` to compile the code to Javascript.

Run `node dist/examples/runFasterCode.js` (to use GraphQL) or `node dist/examples/runOldCode.ts` (to use REST calls), to run the example program. It requires internet access, since it calls the GitHub API. It will take a couple minutes to complete. Some of the output includes the word "ERROR", so don't panic.

## Local testing of the npm packaging

You should have local copies of both the oss-mariner project and the project that will include it.
In the oss-mariner project, run `npm link`. This will "publish" oss-mariner locally on your
computer. Then in the other project, run `npm link oss-mariner`.
This will replace the public npm version of oss-mariner with your local copy.

## Project Maintainers

The [Open Source team at Indeed](https://opensource.indeedeng.io/), who can be reached at opensource@indeed.com.

## How to Publish

If you are a maintainer, you can follow these steps to publish a new version of the package:

1. Create a branch named "publish-x.y.z (x.y.z will be the version number)
1. Update the version number in package.json
1. Be sure the version number in package.json is correct
1. Run `npm install` to update package-lock.json
1. Run `npm run build` and `npm run lint` to make sure there are no errors
1. Submit and merge a PR to bump the version number
1. Login to npm if you haven’t already: `npm login`
1. Do a dry run to make sure the package looks good: `npm publish --dry-run`
1. Publish: `npm publish`
1. Verify: <https://www.npmjs.com/package/oss-mariner>

## Code of Conduct

This project is governed by the [Contributor Covenant v 1.4.1](CODE_OF_CONDUCT.md).

## License

This project uses the [Apache 2.0](LICENSE) license.
