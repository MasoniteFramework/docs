# Environments \(in progress\)

## Introduction

Environments in Masonite are defined in `.env` files and contain all your secret environment variables that should not be pushed into source control. You can have multiple environment files that are loaded when the server first starts. We'll walk through how to configure your environment variables in this documentation.

{% hint style="warning" %}
Never load any of your .env files into source control. `.env` and `.env.*` are in the `.gitignore` file by default so you should not worry about accidentally pushing these files into source control.
{% endhint %}

## Getting Started



