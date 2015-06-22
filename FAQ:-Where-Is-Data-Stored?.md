All your projects are stored on your own computer in the workspace directory listed below. To open it, click "Browse workspace directory" on your OpenRefine application home page, by default at http://127.0.0.1:3333/

**MacOSX:**
* `~/Library/Application Support/OpenRefine/`
* `~/Library/Application Support/Google/Refine/` (older Google Refine versions)
* Logging is to `/var/log/daemon.log` - grep for `com.google.refine.Refine`

**Windows:** depending on the Windows version, the data is in one of these directories
* `C:\Documents and Settings\(user id)\Local Settings\Application Data\OpenRefine`
* `C:\Users\(user id)\AppData\Roaming\OpenRefine`
* `C:\Users\(user id)\OpenRefine`

for older Google Refine releases, replace `OpenRefine` with `Google\Refine`

**Linux:**
* `~/.local/share/openrefine/`