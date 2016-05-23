## Project Setup {#project-setup}

Before we begin with the instruction, we should describe how we expect your projects to be set up.

1.  Create a new Project in Visual Studio (.NET 4.5 or higher)
2.  Right click on “References” in Solution Explorer and select “Manage NuGet Packages…”
3.  Search for “**NBitcoin”** and install it. (Or NBitcoin.Mono on MAC and Linux.)
![](../assets/nuget.png)  

> **Tip:** If you are on MAC or Linux and reference NBitcoin instead of NBitcoin.Mono you will be missing some classes.  

NBitcoin is the .NET Bitcoin library, it is open-source, maintained by Nicolas Dorier, the main author of this book. 
This library should always be included if you do anything Bitcoin related in C#.  
NBitcoin supports cross-platform applications.  

### ## # How to debug into NBitcoin source code (optional)  

NBitcoin lets you debug into its code, to make your life easier. For this feature to work make sure you have source server support enabled in Visual Studio (Tools/Options).   
![](../assets/visualstudio_enablesourceserversupport.png)  

Now, if you step into NBitcoin code, the source code will be automatically fetched on github, and appear in visual studio debugger.  
