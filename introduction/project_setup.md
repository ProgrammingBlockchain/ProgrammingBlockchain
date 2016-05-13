## Project Setup {#project-setup}

Before we begin with the instruction, we should describe how we expect your projects to be set up.

1.  Create a new Console Application in Visual Studio (.NET 4.5 or higher)
2.  Right click on “References” in Solution Explorer and select “Manage NuGet Packages…”
3.  Search for “**NBitcoin”** and install it.  
![](../assets/nuget.png)

NBitcoin is the .NET Bitcoin library, it is open-source, maintained by Nicolas Dorier, the main author of this book.  
This library should always be included if you do anything Bitcoin related in C#.  
NBitcoin supports cross-platform applications.  
NBitcoin lets you debug into its code, to make your life easier. For this feature to work make sure you have source server support enabled in Visual Studio.   
![](../assets/visualstudio_enablesourceserversupport.png)  


