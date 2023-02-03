# Windows Installer Workflows

These workflows center around the basic functions of the Windows Installer.  These include both the act of installing and uninstalling O3DE.

## Common Issues

*   Invalid install location (Directory isn't empty)
*   Install already exists (side-by-side install not currently supported)

## Workflows

| Workflow                         | Steps                                                                                                                                                                            | Expectations                                                                                                                      |
|----------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| Install O3DE In Default Location | 1.  Download latest Release or Development Installer<br>2.  Run the Installer<br>3.  Select "Install"                                                                            | *   O3DE successfully installs in the default location (Generally the root of the C drive)<br>*   User is able to launch o3de.exe |
| Install O3DE in Custom Location  | 1.  Download latest Release or Development Installer<br>2.  Run the Installer<br>3.  Select "Options"<br>4.  Browse to a new location<br>5.  Select "OK"<br>6.  Select "Install" | *   O3DE successfully installs in the user defined location<br>*   User is able to launch o3de.exe                                |
| Uninstall O3DE                   | 1.  With O3DE already installed, launch the installer<br>2.  Select "Uninstall"                                                                                                  | The installer successfully uninstalls O3DE from the previously defined location on disk                                           |
