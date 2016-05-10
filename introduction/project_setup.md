## Project Setup {#project-setup}

Before we begin with the instruction, we should describe how we expect your project to be set up.

1.  Open Visual Studio and create a new Console Application. Name it “ProgrammingBlockchain.”
2.  Right click on “References” in Solution Explorer and select “Manage NuGet Packages…”
3.  Search for “**NBitcoin”** and install it. Note: The information provided in the image is for reference only. Actual version and publication dates may change as you are reading this.

1.  Right click on “ProgrammingBlockchain” in the Solution Explorer and select “Add” then “New Folder.” Name the folder “Chapters.”
2.  Right click “Chapters” and select “Add” then “New Class.” Name this class “Chapter1.” You will do this for every new chapter in the book.
3.  Open “Program.cs” and add the following code:

1.  Note “using ProgrammingBlockchain.Chapters;” was added to the using block.
2.  At this point Visual Studio is complaining that “chapter.Lesson4();” does not exist! Keep reading and we’ll create it.