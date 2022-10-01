# Simple Blogsite

This sample project demonstrates how to make a simple blogsite that can be deployed on the [Internet Computer](https://github.com/dfinity/ic). It is a minimal design that allows users to authenticate with a wallet and then create, read, update and delete posts.

You can experiment with a live running version of this project on this [link](https://gaqra-oqaaa-aaaah-qc4qa-cai.ic0.app/).

We are going to go through the most crucial parts in this text. The full code of this project can be found in [this repository](https://github.com/lukasvozda/icblog). You are welcome to clone the repository and experiment with the code on your own. 

## Backend

The project consist of a backend written in **Motoko programming language**, that has been designed specifically for writing smart contracts on the Internet Computer Protocol. You can find backend code in the `canisters/blog/main.mo` file.

Let's go through some important parts of the code.

### Post type
The Post type defines the structure of a blog post. If you add other attributes to it, do not forget to update create and update functions as well.

```
type Post = {
    title : Text;
    time_created: Time.Time;
    time_updated: Time.Time;
    content : Text;
    description : Text;
    published : Bool;
    author : Principal;
    tags : [Text]
};
```

### HashMap
HashMap is our database that stores posts each under its own hash (we use a simple natural number PostId as a hash). This way we can reach individual posts very effectively.
```
private var blogposts = Map.HashMap<PostId, Post>(0, eq, Hash.hash);
```
Keep in mind **HashMap is not a stable storage**. This means that it does not sustain posts between updates. That is why we also defined a postupgrade and preupgrade methods that will take care of data between updates.

### Create a post
Here is a sample of a create function. It firstly checks whether an user is authenticated – we do not allow anonymous principals to create a post. Also, we do not want empty titles. If it passes, a postId is assigned and a new post can be created. Otherwise an #err type is returned. Specific #err messages are handled in the front end to give proper feedback to the user.

```
public shared(msg) func create(post : {title : Text; description : Text; content : Text; published : Bool; tags : [Text]}): async Result.Result<(),Error> {
    if(Principal.isAnonymous(msg.caller)){ // Only allows signed users to create a posts
        return #err(#UserNotAuthenticated); // If the caller is anonymous Principal "2vxsx-fae" then return an error
    };

    // Title is required in the fron-tend, but we can also check in the backend
    if(post.title == ""){ 
        return #err(#EmptyTitle) 
    };

    let postId = next;
    next += 1; // increment the counter so we never try to create a post under the same index

    let blogpost: Post = {
        time_created = Time.now(); // Time is assigned on the backend side
        time_updated = Time.now();
        title = post.title; 
        description = post.description;
        content = post.content;
        published = post.published;
        author = msg.caller; // Author is assigned on the backend side too
        tags = post.tags;
    };
    
    blogposts.put(postId, blogpost);
    return #ok(()); // Return an OK result
};
```
Notice that the function receives only a simplified version of a post. That is because time_created, time_updated and author attributes of the post are assigned in the backend and we do not want them to be manipulated by the user.

Other functions from the CRUD interface follow the same pattern. Check the full code in the `canisters/blog/main.mo` file. There are also functions for listing, filtering and sorting posts.

## Frontend

Front-end is written using the Svelte compiler. There are Routes for Create, Read, Update, Delete and List posts. You can find it in the `frontend/routes` directory.

Svelte starter app created by [Mio Quispe](https://github.com/MioQuispe) is used in this project, he also created [starter apps for React, Vue or TS](https://github.com/MioQuispe/create-ic-app). If you want to use a different toolkit, you can follow this repository. This starter app also utilises the [Connect2IC toolkit](https://github.com/Connect2IC/connect2ic) that simplifies the process of wallet connection for different wallet providers.

## Local deployment

 We assume that you have:
- [Dfnity SDK](https://internetcomputer.org/docs/current/developer-docs/quickstart/hello10mins) installed
- NodeJS installed

Once you have cloned the [repository](https://github.com/lukasvozda/icblog), follow this process in your terminal:

1. In your project directory, run this command to install JS dependencies:
```
$ npm install
```
2. Start local Internet Computer replica (or open a new terminal window and run it without the --background parameter):
```
$ dfx start --background 
```
3. Deploy your canisters locally:
```
$ dfx deploy
```
4. Run local dev server:
```
$ npm run dev
```
You should see a localhost URL looking like this "Local: http://localhost:3000/" in your terminal. Open this in your web browser and see the app running.

## Deploy to the mainnet

If you have working local development replica, you can deploy your project to the mainnet by running this command:
```
$ dfx deploy --network ic
```
You are going to need a cycles wallet. Go through [this tutorial](https://internetcomputer.org/docs/current/developer-docs/quickstart/network-quickstart) to make it working.