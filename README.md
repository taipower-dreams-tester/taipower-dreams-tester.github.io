# taipower-dreams-tester.github.io

## Using Slate in Docker

https://github.com/slatedocs/slate/wiki/Using-Slate-in-Docker

### Dependencies

* [Docker](https://www.docker.com/)

### Getting Started

Grab the slate image:

```sh
docker pull slatedocs/slate
```

### Building Slate

To use Docker to just build the site, run:

```
docker run --rm --name slate -v $(pwd)/build:/srv/slate/build -v $(pwd)/source:/srv/slate/source slatedocs/slate build
```

After this command completes, you should see the built artifacts for your site in the `$(pwd)/build` directory, which you can then statically serve for your website.

### Running Slate

To run the development server for Slate to aid in working on the site, run:

```
docker run --rm --name slate -p 4567:4567 -v $(pwd)/source:/srv/slate/source slatedocs/slate serve
```

and you will be able to access the site at http://localhost:4567 until you stop the running container process.


## Deploying Slate

https://github.com/slatedocs/slate/wiki/Deploying-Slate

### Publishing The Docs to GitHub Pages

1. Build the site. ([Building Slate](#building-slate))
2. Check the working repo: `git remote show origin`.
3. Run `./deploy.sh --push-only`

The changes should now be live on GitHub Pages.

Note that if this is your first time publishing Slate, it can sometimes take ten minutes or so before your content is available online. It can also take a moment even if it's not the first time. 


## Useful Resources

* Slate Wiki : https://github.com/slatedocs/slate/wiki
