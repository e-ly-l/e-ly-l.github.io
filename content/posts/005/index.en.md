+++
title = "Repository Structure in a Sole Nx Experimental Module"
description = ""
summary = ""
date = 2024-08-02T23:45:00+08:00
slug = "experimental-module-in-nx-repository"
tags = ["monorepo","nx", "experiment", "typescript"]
categories = ["implement"]
+++

## Preface

Some of my readers already know that I'm a scraper engineer in my daily job. My team migrated from a monolithic to a monorepo architecture almost 10 months ago, and I still believe it was the best decision we made last year. This change means we now have many modules in a single repository.

Recently, I received a requirement to research scraping data from a new platform. Consequently, I need several modules as follows:

1. **types:** Define raw, processed, and aggregated data types from the platform.
2. **model:** Define the model to store the data in the database.
3. **scraper:** Define the scraper API & SDK to get data from the platform.
4. **worker:** Define the worker for distributed, cloud-based, and scalable scraping.

However, if I create these modules in the same repository initially, I need to change the code everywhere. This will not only make the codebase messy but also make the code review process more difficult, increasing the likelihood of merge conflicts.

So, how can we make a single module serve as both a library that can be imported by other modules and an runnable application at the same time?

## Repository Structure

### Endpoints of Library & Application

Firstly, we need two different endpoints for the `library` and the `application`. The `library` endpoint is the entry point of the module, which can be imported by other modules. We need to ensure that declaration files are generated for the `library` endpoint.

The `application` endpoint is the entry point of the module, which can be executed as a standalone application. We need to bundle all imports and dependencies into a single file for execution.

The recommended repository structure is as follows:

```bash
scraper
├── data # static files
├── experiments # research
├── scripts # manual scripts, e.g. publish pkg, deploy app
├── src # main application; included solely in `tsconfig.app.json`
│   ├── index.ts # entry point as library
│   └── main.ts # worker as application
└── test # testing
```

### tsconfig

We need several tsconfig files for different purposes.

#### tsconfig for Application

All modules should temporarily be put in the `src` folder, and the endpoint for the application should be `main.ts`.
We can exclude `index.ts` from the application endpoint because it’s only for the library.

```json
// tsconfig.app.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "outDir": "../../../dist/out-tsc",
    "module": "commonjs",
    "types": ["node"]
  },
  "exclude": [
    "jest.config.ts",
    "src/**/*.spec.ts",
    "src/**/*.test.ts",
    "src/index.ts"
  ],
  "include": ["src/**/*.ts"]
}
```

#### tsconfig for library

In contrast to the application, we need to exclude `main.ts` from the library endpoint because it’s only for the application. Remember to set `declaration` to true to generate declaration files for type exports.

```json
// tsconfig.lib.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "outDir": "../../../dist/out-tsc",
    "declaration": true,
    "types": ["node"]
  },
  "include": ["src/**/*.ts"],
  "exclude": [
    "jest.config.ts",
    "src/**/*.spec.ts",
    "src/**/*.test.ts",
    "src/main.ts"
  ]
}
```

#### tsconfig for development

I also create a `tsconfig.dev.json` for research purposes. I put all the experiments in the `experiments` folder and `scripts` in the scripts folder, so the application and library won’t be affected.

```json
// tsconfig.dev.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "outDir": "../../../dist/out-tsc",
    "module": "commonjs",
    "types": ["node"]
  },
  "exclude": ["jest.config.ts", "src/**/*.spec.ts", "src/**/*.test.ts"],
  "include": ["src/**/*.ts", "experiments/**/*.ts", "scripts/**/*.ts"]
}
```

#### tsconfig for test

I haven’t written serious tests for the module yet, but I will put the test files in the `test` folder for the same reasons as `experiments` and `scripts`.

```json
// tsconfig.spec.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "outDir": "../../../dist/out-tsc",
    "module": "commonjs",
    "types": ["jest", "node"]
  },
  "include": [
    "jest.config.ts",
    "test/**/*.ts",
    "src/**/*.test.ts",
    "src/**/*.spec.ts",
    "src/**/*.d.ts"
  ]
}
```

#### tsconfig in module

The `tsconfig.json` in the module is the base configuration for all other tsconfig files, so we need to import all the above tsconfig files here to activate the configuration.

```json
// tsconfig.json
{
  "extends": "../../../tsconfig.base.json",
  "files": [],
  "include": [],
  "references": [
    {
      "path": "./tsconfig.app.json"
    },
    {
      "path": "./tsconfig.dev.json"
    },
    {
      "path": "./tsconfig.lib.json"
    },
    {
      "path": "./tsconfig.spec.json"
    }
  ],
  "compilerOptions": {
    "esModuleInterop": true
  }
}
```

#### tsconfig in Root (Nx Workspace)

As an Nx repository, we need to add the module to the paths in the `tsconfig.base.json` in the root. Otherwise, we cannot import this module in other modules.

My team sets the paths in `tsconfig.base.json` in a format like `organization/module_name` to make it more readable. This makes it easy when we publish the module as a package, as we can use the same name as the package name.

```json
// tsconfig.base.json
{
  ...
  "paths": {
    "organization/mododule_name": ["path/to/module/src/index.ts"]
  }
}
```

### Module Project Configuration

This part is somewhat complicated, so I need to explain some basic knowledge of Nx first. As mentioned above, formally, we should separate the `library` and `application` modules in the repository. In that way, we need to generate them with `@nx/node:library` and `@nx/node:application`. However, for the early stages of development, we can just use `@nx/node:library` to generate the module and then modify the configuration manually. This means I need to add the application configuration to the module project.

#### Build Configuration for Library

This part is for other modules to import the module as a package. We can keep the configuration after using the `@nx/node:library` generator. However, we can specify `tsConfig` as `tsconfig.lib.json` in `project.json` to include only necessary source files and the declaration files.

```json
// project.json
{
  "targets": {
    ...
    "build": {
        "executor": "@nx/esbuild:esbuild",
        "outputs": ["{options.outputPath}"],
        "options": {
          "format": ["cjs"],
          "outputPath": "dist/path/to/module",
          "main": "path/to/module/src/index.ts",
          "tsConfig": "path/to/module/tsconfig.lib.json",
          "generatePackageJson": true
        }
      }
  }
}
```

#### Build Configuration for Application

We need to add `build-main` configuration to the module project to build the application. Note the compile fields, e.g., `bundle`, `thirdParty`, etc. We need to specify the main field to the entry point of the application and `tsconfig` to the `tsconfig.app.json` to include only necessary source files.

If we want to define some `deploy` workflow here, we can add deploy configuration to the module project.

```json
// project.json
{
  "targets": {
    ...
    "build-main": {
      "executor": "@nx/esbuild:esbuild",
      "outputs": ["{options.outputPath}"],
      "defaultConfiguration": "production",
      "options": {
        "platform": "node",
        "outputPath": "dist/path/to/module",
        "format": ["cjs"],
        "main": "path/to/module/src/main.ts",
        "tsConfig": "path/to/module/tsconfig.app.json",
        "assets": ["path/to/module/src/assets"],
        "generatePackageJson": true,
        "bundle": true,
        "thirdParty": true,
        "esbuildOptions": {
          "sourcemap": true,
          "outExtension": {
            ".js": ".js"
          }
        }
      },
      "configurations": {
        "development": {},
        "production": {
          "esbuildOptions": {
            "sourcemap": false,
            "outExtension": {
              ".js": ".js"
            }
          }
        }
      }
    },
    "deploy": {
      "dependsOn": ["build-main"],
      "executor": "nx:run-commands",
      "options": {
        "commands": ["sh path/to/module/scripts/deploy.sh"]
      }
    }
  }
}
```

Eventually, the `project.json` should look like this:

```json
// project.json
{
  "name": "module_name",
  "$schema": "../../../node_modules/nx/schemas/project-schema.json",
  "sourceRoot": "path/to/module/src",
  "projectType": "library",
  "targets": {
    "build": {
      "executor": "@nx/esbuild:esbuild",
      "outputs": ["{options.outputPath}"],
      "options": {
        "format": ["cjs"],
        "outputPath": "dist/path/to/module",
        "main": "path/to/module/src/index.ts",
        "tsConfig": "path/to/module/tsconfig.lib.json",
        "generatePackageJson": true
      }
    },
    "build-main": {
      "executor": "@nx/esbuild:esbuild",
      "outputs": ["{options.outputPath}"],
      "defaultConfiguration": "production",
      "options": {
        "platform": "node",
        "outputPath": "dist/path/to/module",
        "format": ["cjs"],
        "main": "path/to/module/src/main.ts",
        "tsConfig": "path/to/module/tsconfig.app.json",
        "assets": ["path/to/module/src/assets"],
        "generatePackageJson": true,
        "bundle": true,
        "thirdParty": true,
        "esbuildOptions": {
          "sourcemap": true,
          "outExtension": {
            ".js": ".js"
          }
        }
      },
      "configurations": {
        "development": {},
        "production": {
          "esbuildOptions": {
            "sourcemap": false,
            "outExtension": {
              ".js": ".js"
            }
          }
        }
      }
    },
    "deploy": {
      "dependsOn": ["build-main"],
      "executor": "nx:run-commands",
      "options": {
        "commands": ["sh path/to/module/scripts/deploy.sh"]
      }
    }
  },
  "tags": []
}
```

If we want to define more customized dependency rules, Nx provides a way to configure them in `.eslintrc.json`[^1].

[^1]: [Nx Docs - Enforce Project Dependency Rules](https://nx.dev/concepts/decisions/project-dependency-rules)
