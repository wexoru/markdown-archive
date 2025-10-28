# Guide to Building a Roblox Internal DLL like BlueFire

**Disclaimer:** This guide is provided for educational and informational purposes only. It is intended to demonstrate programming concepts, software building processes, and integration techniques in C++ and .NET environments. The author does not endorse or encourage any form of cheating, exploitation, or violation of Roblox's terms of service, game integrity, or any laws. Misuse of this information, including creating or distributing cheats for online games, may result in account bans, legal consequences, or other penalties. Proceed at your own risk and ensure all actions comply with applicable laws and platform policies.

## Introduction

This guide outlines the steps to build a custom Roblox Internal DLL similar to "BlueFire," which is typically used for injecting scripts or modifying game behavior in Roblox. The process assumes you have access to a source code project (e.g., a C++ DLL project in Visual Studio) that resembles BlueFire's structure. You'll need to update offsets and shuffles based on the latest Roblox version, rename the project, build it, and integrate it into .NET applications like Windows Forms or WPF.

Prerequisites:
- Visual Studio 2022 (or later) with C++ and .NET workloads installed.
- Basic knowledge of C++, offsets in memory hacking, and .NET (C#).
- Roblox installed for testing (use responsibly).
- Source code for a similar DLL project (not provided here; assume you have it).

## Step 1: Installing OpenSSL for C++

OpenSSL is often required for cryptographic operations in such projects (e.g., handling encrypted states or communications). Install it via vcpkg (recommended for ease) or manually from the official site.

### Option 1: Using vcpkg (Recommended)
1. Install vcpkg if not already done:
   - Clone the repository: `git clone https://github.com/microsoft/vcpkg.git`
   - Bootstrap: `.\vcpkg\bootstrap-vcpkg.bat`
   - Integrate with Visual Studio: `.\vcpkg\vcpkg integrate install`

2. Install OpenSSL:
   - Run: `.\vcpkg\vcpkg install openssl:x64-windows` (for 64-bit; adjust for your architecture).

3. In Visual Studio:
   - Open your project.
   - Go to Project > Properties > C/C++ > General > Additional Include Directories: Add `$(VCPKG_ROOT)\installed\x64-windows\include`.
   - Linker > General > Additional Library Directories: Add `$(VCPKG_ROOT)\installed\x64-windows\lib`.
   - Linker > Input > Additional Dependencies: Add `libcrypto.lib;libssl.lib`.

### Option 2: Manual Installation from Official Site
1. Download the latest OpenSSL from https://www.openssl.org/source/ (e.g., openssl-3.x.x.tar.gz).
2. Extract and build:
   - Install Perl (e.g., Strawberry Perl) and NASM (Netwide Assembler).
   - Run in Command Prompt: `perl Configure VC-WIN64A` (for 64-bit).
   - Then: `nmake` and `nmake install`.
3. Copy the `include` and `lib` folders to your project or a system path.
4. Update Visual Studio properties as in Option 1, pointing to your OpenSSL installation paths.

Test by including `<openssl/evp.h>` in your code and compiling a simple test file.

## Step 2: Updating Offsets and Shuffles

Offsets and shuffles are memory addresses and macro definitions that point to specific functions or data in Roblox's executable. They change with Roblox updates, so you must replace them in your project's `Offsets.hpp` and `Shuffles.hpp` files using the provided data.

### From Provided Files
- `! OFFSETS&SHUFFLES.txt`: Contains older offsets (e.g., for version-4aeb17bd13994560) and shuffles. Use this as a reference for structure.
- `Shuffles.txt`: Updated shuffles for version-b591875ddfbc4294.
- `Offsets.txt`: Updated offsets for version-b591875ddfbc4294.

### Replacement Process
1. Open your project's `Offsets.hpp` (or similar file).
   - Copy offsets from `Offsets.txt` (e.g., `const uintptr_t Print = REBASE(0x14CD600);`).
   - Replace all old offsets with new ones. Ensure `REBASE(x)` macro is defined correctly (e.g., `#define REBASE(x) x + (uintptr_t)GetModuleHandle(nullptr)`).
   - Update namespaces like `Update`, `RBX`, etc., with new values for functions like `LuaVM__Load`, `Task__Defer`, etc.
   - For DataModel, Instance, etc., update constants like `Name = 0x68;`.

2. Open `Shuffles.hpp` (or similar).
   - Copy shuffle macros from `Shuffles.txt` (e.g., `#define LUAU_SHUFFLE3(s, a1, a2, a3) a2 s a3 s a1`).
   - If your project uses older shuffles from `! OFFSETS&SHUFFLES.txt` (e.g., `#define LUAU_SHUFFLE3(s, a1, a2, a3) a3 s a2 s a1`), replace them entirely with the new ones.
   - Ensure separators like `LUAU_COMMA_SEP` and `LUAU_SEMICOLON_SEP` match.

3. If your project has additional shuffles (e.g., for Proto, LSTATE), integrate from `! OFFSETS&SHUFFLES.txt` like `#define PROTO_MEMBER1_ENC VMValue0`.

4. Verify:
   - Compile the project to check for errors.
   - Use a tool like Cheat Engine to validate offsets in a running Roblox process.

Note: Offsets are version-specific. For the current date (October 28, 2025), use the latest from reliable sources or dump them yourself using tools like Hyperion dumper.

## Step 3: Renaming All "BlueFire" References in Visual Studio

To rebrand the DLL (e.g., to "MyCustomDLL"):
1. Open the solution in Visual Studio.
2. Use "Find and Replace" (Ctrl + H):
   - Find: "BlueFire" (case-sensitive, whole word).
   - Replace: "MyCustomDLL".
   - Scope: Entire Solution.
   - Include: All file types (e.g., .cpp, .hpp, .csproj if mixed).

3. Rename files:
   - Right-click project > Rename.
   - Update .sln, .vcxproj, and output DLL name in Properties > Application > Assembly Name/Output File.

4. Update namespaces, class names, and strings in code (e.g., API endpoints like "http://localhost:1337/Bluefire/Scheduled" to your new name).
5. Clean and rebuild to ensure no leftovers.

## Step 4: Building the Project

1. Open the project in Visual Studio.
2. Set configuration: Release > x64 (for Roblox compatibility).
3. Resolve dependencies: Ensure OpenSSL is linked, and any other libs (e.g., Windows.h).
4. Build: Build > Build Solution (Ctrl + Shift + B).
   - Output: A .dll file (e.g., MyCustomDLL.dll) in the bin/Release folder.
5. Test: Inject into Roblox using a simple injector (not covered here; use responsibly).

Common issues:
- Linker errors: Check library paths.
- Compiler errors: Fix offset types (e.g., uintptr_t vs. uint64_t).

## Step 5: Using in Windows Forms and WPF Applications via API

Once built, integrate the DLL into a .NET app for injection and execution. Assume your DLL exposes an API (e.g., via HTTP server on localhost:1337 for script scheduling).

### In Windows Forms or WPF (C#)
1. Create a new .NET project (Windows Forms App or WPF App).
2. Add references: System.Net.Http, Newtonsoft.Json.
3. For attachment (injection):
   - Use Process.Start to run an injector executable bundled with your app.
   ```csharp
   using System.Diagnostics;
   using System.IO;

   // In a button click event or method
   Process.Start(Path.Combine(Environment.CurrentDirectory, "Main\\Injector.exe"));
   ```

4. For execution (sending script to the DLL's API):
   - Assume your editor is a TextBox (e.g., this.editor.Text).
   ```csharp
   using System.Net.Http;
   using System.Text;
   using Newtonsoft.Json;
   using System.Windows; // For MessageBox in WPF

   // In an async button click event
   private async void ExecuteButton_Click(object sender, RoutedEventArgs e) // WPF example
   {
       StringContent content = new StringContent(JsonConvert.SerializeObject(new { Script = this.editor.Text }), Encoding.UTF8, "application/json");
       try
       {
           using (HttpClient Client = new HttpClient())
           {
               HttpResponseMessage httpResponseMessage = await Client.PostAsync("http://localhost:1337/MyCustomDLL/Scheduled", content); // Update endpoint
           }
       }
       catch (Exception ex)
       {
           MessageBox.Show("Request failed: " + ex.Message);
       }
   }
   ```

5. Run the app: Build and execute. Attach first, then send scripts.

This setup allows controlling the injected DLL via HTTP for security/separation.

## Conclusion

You've now built and integrated a custom Roblox Internal DLL. Remember, this is for learning—always respect game developers and communities. For updates, monitor Roblox versions and redump offsets as needed.

## Credits:
This was written by me (0Zayn), and Ethan (ethanmcdonagh), and given the code name Ballistic, many upcoming new executors have been using this module, including but not limited to Artemis, Athena, Atlantis, and Protoware (NOTE: Protoware did not even have our permission, and claimed it was theirs).
Due to all this I have decided to leak the source to hopefully put a stop, and be able to have credit for our work.

Please feel free to do whatever with this, just not distribute it as your own product

- Zayn, and Ethan

Contributors:

- Entity (Entitynt)
- Joe (JoeIsGod)

(NOTE: THIS PROJECT HAS A LICENSE)

## Some info about this source code:
ballistics fork
originaly skidded by ronoploda and got leaked by igromanvtv
that is not my work lol
