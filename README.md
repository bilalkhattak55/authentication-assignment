1================================;
npx create-next-app@latest
say yes to every thing;
install dependecies;
npm i axios bcryptjs jsonwebtoken nodemailer react-hot-toast mongoose

2==================================;
create login, signup and profile folder in app folder and page.tsx in all of these files;

3===================================;
create api foler and user folder in it;
then, login and signup folder in user folder;
then, route.ts in sign up and login folder;


4=====================================;
in signup folder page.tsx;
"use client";
import Link from "next/link";
import React from "react";
import { useRouter } from "next/navigation";
import axios from "axios";

export default function SignupPage() {
    const [user, setUser] = React.useState({
        email: "",
        password: "",
        userName: ""
    });

    const onSignup = async ()=> {}
    return (
        <div className="flex flex-col items-center justify-center min-h-screen py-2">
            <h1>SignupPage</h1>
            <hr />
            <label htmlFor="username">username</label>
            <input
            className="p-2 border border-gray-300 rounded-lg mb-4 focus:outline-none focus:border-gray-600"
            type="text"
            id="username"
            value={user.userName}
            onChange={(e)=> setUser({...user, userName: e.target.value})}
            placeholder="username"
            />
            <label htmlFor="email">email</label>
            <input
            className="p-2 border border-gray-300 rounded-lg mb-4 focus:outline-none focus:border-gray-600"
            type="email"
            id="email"
            value={user.email}
            onChange={(e)=> setUser({...user, email: e.target.value})}
            placeholder="email"
            />
            <label htmlFor="password">password</label>
            <input
            className="p-2 border border-gray-300 rounded-lg mb-4 focus:outline-none focus:border-gray-600"
            type="password"
            id="password"
            value={user.password}
            onChange={(e)=> setUser({...user, password: e.target.value})}
            placeholder="password"
            />
            <button
            onClick={onSignup}
            className="p-2 border border-gray-300 rounded-lg mb-4 focus:outline-none focus:border-gray-600">Signup here</button>
            <Link href={`/login`}>Visit Login page</Link>
        </div>
    )
}


then, 
copy the above code and paste in login folder page.tsx;
delete username fields and make some important changes;



5===================================================;
make a [id] folder in profile folder and page.tsx in [id] folder;

export default function UserProfilePage({params}: any) {
    return(
        <div className="flex flex-col items-center justify-center min-h-screen py-2">
            <h1>Profile</h1>
            <hr />
            <p className="text-4xl">profile details
            <span className="p-2 rounded bg-orange-500 text-black">{params.id}</span> </p>

        </div>
    )
}


6======================================================;
make dbConfig, helpers, models folder in main src folder;

make a dbConfig.ts file in dbConfig folder;
import mongoose from "mongoose";

export async function connect() {
    try {
         mongoose.connect(process.env.MONGO_URL!);
         const connection = mongoose.connection;

         connection.on('connected', ()=> {
            console.log("mongodb is connected successfully");
         });

         connection.on('error', (err)=> {
            console.log('Mongodb connection error, please make sure mongoDB is running' + err);
            process.exit()
         })
    }catch(error) {
        console.log("something goes wrong")
        console.log(error)
    }
}

then,
add this logic to signup page;
const [buttonDisabled, setButtonDisabled] = React.useState(false)
useEffect(()=> {
        if(user.email.length > 0 && user.password.length > 0 && user.username.length > 0) {
            setButtonDisabled(false);
        } else {
            setButtonDisabled(true);
        }
    }, [user])

{buttonDisabled ? "No Signup" : "Signup"}</button>

then,

const onSignup = async ()=> {
        try {
            setLoading(true)
            const response = await axios.post("/api/users/signup", user)
             console.log("Signup success", response.data)
             router.push("/login")
        }catch(error: any) {
            toast.error(error.message)
         console.log(error.meassage)
        }finally{
            setLoading(false)
        }
    };


then,
check sign up and check network and database for userData;



7=============================================;
create login route;
import {connect} from "@/dbConfig/dbConfig";
import User from "@/models/userModel";
import { NextRequest, NextResponse } from "next/server";
import bcryptjs from "bcryptjs";
import jwt from "jsonwebtoken"


connect();

export async function POST(request: NextRequest) {
    try {
        const reqBody = await request.json();
        const {email, password} = reqBody;
        console.log(reqBody);

        //check if the user exist;
        const user = await User.findOne({email});
        if(!user) {
            return NextResponse.json({error:"user does not exist"}, {status: 500});
        }
        //comparing passwords;
        const validPassword = await bcryptjs.compare(password, user.password);

        if(!validPassword) {
            return NextResponse.json({error: "Invalid password"}, {status: 400})
        }

        //create token data;
        const tokenData = {
            id: user._id,
            username: user.username,
            email: user.email
        }

        //create token,
        const token = await jwt.sign(tokenData, process.env.TOKEN_SECRET!, {expiresIn: "1d"})

        const response = NextResponse.json({
            message: "login successfully",
            success: true
        })
        response.cookies.set("token", token, {
            httpOnly: true
        })

        return response;

    }catch(error: any) {
        return NextResponse.json({error: error.message},{status: 500});
        
    }
}



8==========================================;
now create login page;
const router = useRouter();

    const [user, setUser] = React.useState({
        email: "",
        password: "",  
    });

    const [buttonDisabled, setButtonDisabled] = React.useState(false)
    const [loading, setLoading] = React.useState(false)

    const onLogin = async ()=> {
        try {
          setLoading(true)
          const response = await axios.post("/api/users/login", user)
          console.log("Login success", response.data)
          toast.success("Login success")
          router.push("/profile")
        }catch(error: any) {
            console.log("login failed", error.message);
            toast.error(error.message);  
        }finally {
            setLoading(false)
        }
    }

    useEffect(()=> {
        if(user.email.length > 0 && user.password.length > 0) {
            setButtonDisabled(false)
        }else {
            setButtonDisabled(true)
        }
    }, [user])



9=================================;
logout functionalty;
import { NextResponse } from "next/server";

export async function GET() {
    try {
        const response = NextResponse.json({
            message: "Logout successfully",
            success: true
        })
        
        response.cookies.set("token", "", {httpOnly: true, expires: new Date(0)});

        return response;

    }catch(error: any) {
        return NextResponse.json({error: error.message}, {status: 500})
    }
}


10=============================;
now come to profile page and make logout button there;




























