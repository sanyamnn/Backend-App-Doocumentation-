## Let's document the Tasks in Backend of our app .
1. To begin with , let's start with the first file of our project that is "package.json" Apart from obvious , package.json has some scripts that you may find different depending on the project name and type . Let's dicuss them in detail  
    1. `"start:ts": "nodemon --watch 'src/**/*.ts' --ignore 'src/**/*.spec.ts' --exec 'ts-node' src/beamer.ts"`  
       in this command , start:ts is defined as the command that will expecute the following depending on instructions provided : `nodemon --watch .. .. ` is to used to monitor any real time changes in the app and nodemon automatically recompiles and runs your application on the same/new port depending upon availability. ` --exec ..` used to mention execution command for the main file of app following the path of file.
    2. ` "start": "npm-run-all --parallel start:server lint:server:watch" `  
        In this command , npm-run-all is used to run multiple npm scripts parellt and other commands are also present to start server and enable watching that seem obvious.
2. Now , if you had seen closely in previous point that execution file (first access file) of the app was mentioned to be ` src/beamer.ts` . Let's make further maps of project based on that.  
  First few statements are used to import bindings used by another module by using statements similar to `import defaultExport as abcImport from "module-name"; ` , like ,  ` import * as http from 'http';` creates an object named http in the code  then, the interpreter will look for each possible import in http module and append it, one by one, to the http object in the code.  
    Let's focus on some important imported modules and their uses , one by one.
      1. ` import { Response, Request, NextFunction,} from 'express'; `  
  In this statement , Response , Request and NextFunction are imported from 'express' middleware . How they work will be understood in greater detail when we discuss about controllers later in this documentation but An express application is a series of middleware function calls . During req-res cycle , middlewares have access to req,res and next middleware to be dealt with . The import structure differs when more functionalities are involved like "err" along with req,res and next when error handling is dealt .
     2. Similarly , in next statement , app.config is imported that contains some predefined constants/variable for the app .
     3. Now , in ` import { app } from './initServer';` , the structure of initServer can be broken into 3 simple parts , * Containing Imports (Third Party , as well as Project Imports) , * Introducing/Declaring middlewares and some other basic App functionalities , * Exporting .  
    In the first part , some basic imports include ejs(as templating engine for project),'logger' that is a request logger middleware from 'morgan', 'compression' to compress request data size , we will discuss multer and cors as we proceed with middleware part of this file.  
    At first,let's discuss multer, multer is used basically for uploadingfiles, multer adds a body object and a file/files object to the req object . The body object contains the values of the text fields of the form, the file or files object contains the files uploaded via the form. like ,in 
    ```javascript
    const storage = multer.diskStorage({
    destination: function (req, file, cb) {
    cb(null, 'public/uploads')
    },
    filename: function (req, file, cb) {
    const ext = file.mimetype.split('/')[1];
    cb(null, `${file.fieldname}-${Date.now()}.${ext}`);
    }
    })
    export const upload = multer({ storage });
    ```
       ¸ here , destination , mentions the path of folder where file is to be stored and filename mentions how the file should be named , while being stored in the folder.  
      Now, for CORS (Cross-Origin-Resource Sharing) , we are using npm-cors middleware
      ```javascript
      const whitelist = [`${CLIENT_URL}:${CLIENT_PORT}`, `${CLIENT_URL}:${PORT}`, `${CLIENT_URL_MOBILE}:${ADMIN_PORT}`, `${CLIENT_URL}:${ADMIN_PORT}`, `${CLIENT_URL}:3000`, `${CLIENT_URL_MOBILE}:3000`, `${CLIENT_URL_MOBILE}:${AUTHOR_PORT}`];
    const corsOptions = {
    origin: (origin: any, callback: Function) => {
    console.log('origin ', origin);
    let isOriginInWhitelist = whitelist.indexOf(origin) !== -1;
    if (!origin) {
      isOriginInWhitelist = true;
    } //same origin
    console.log('in whitelist', isOriginInWhitelist);
    if (isOriginInWhitelist) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by cors'));
    }
    },
    optionsSuccessStatus: 200,
    credentials: true
    };
    app.use(cors(corsOptions));
      ```
      In this , the first stament marks the whitelist address as the address to provide access to resources (because of data being confidential).  
        Before giving actual permission , it checks if origin is in whitelist, if true , it returns a callback taking app further and allowing resource sharing, else returns callback with the error message.,denying permission .
    This was it about initServer..
   4. Now, moving on to next import in beamer.ts , i.e.
   ```javascript
   import httpRoutes from './routes/index';
   ```
   This is to import httpRoutes (primary routes of app) from a file that exports a basic class named , ```HttpRoutes``` and has a declaration of routing url path along with the controller assigned for the task in app.post(,) , in the class's constructor for instant invoke like in ```app.post('/user/su', userController.addUser);``` ,/users/su is the url path and userController is the controller and addUser is a method that is called . It is now important to look at the structure of a controller file , like if we'll go to applicationController.ts or any other controller, we'll see a bunch of methods being declared with async-await calls and promises with parameters , "req,res and next" that work as mentioned above in express middleware discussion (as request object,response object and moving next iteratively)
   (I suggest you to look at declaration of controllers and routes one by one and reading methods)
   5. Moving further in beamer.ts , app.use (with basic express parameters -err,req,res and next), handles errors , with all errors being present in JSON format in errorTypes module (that is imported on top) ,
   Then the server is invoked and is run by app.
