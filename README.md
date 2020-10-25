Introduction
------------

this module contains file handling functions to implement in a Python project.

## Installation
Install via pip:

    pip install os-file-handler


## Usage       
Require FileHandler:
        
    import ostools.file_handler as fh
    

## Functions and signatures:
    
# FileHandler
    
    # will return the content of a directory (full paths)
    def get_dir_content(dir_path, ignored_files_arr=['.DS_Store']):
        f = []
        for path, dirs, files in os.walk(dir_path):
            for filename in files:
                f.append(os.path.join(path, filename))
            break
    
        f = list(filter(lambda x: get_file_name_from_path(x) not in ignored_files_arr, f))
        return f
    
    
    # will return the name of the last dir name from a path
    def get_dir_name(dir_path):
        return os.path.basename(os.path.normpath(dir_path))
    
    
    # will split path to arr of dirs
    def split_path(path):
        path = os.path.normpath(path)
        return path.split(os.sep)
    
    
    # will return the extension of a file from a file path
    def get_extension_from_file(file):
        _, file_extension = os.path.splitext(file)
        return file_extension
    
    
    # will return the file name from a file path
    def get_file_name_from_path(file):
        import ntpath
        return ntpath.basename(file)
    
    
    # will remove a directory
    def remove_dir(path):
        if (os.path.isdir(path)):
            shutil.rmtree(path)
    
    
    # will copy a directory
    def copy_dir(src, dst):
        from distutils.dir_util import copy_tree
        copy_tree(src, dst)
    
    
    # will copy a file to a dest
    def copy_file(src, dst):
        shutil.copy(src, dst)
    
    
    # will copy a list of files to a dir
    def copy_list_of_files(files_list, dst):
        for file in files_list:
            copy_file(file, dst)
    
    
    # will search for a file in a path
    def search_file(path_to_search, file_name):
        from pathlib import Path
        files = []
        for filename in Path(path_to_search).glob('**/' + file_name):
            files.append(filename)
        return files
    
    
    # will replace a text in a line in a file with other line
    def replace_line_for_line(file, line_in_line_dict):
        lines = []
        with open(file, "r") as f:
            for line in f:
                appended = False
                for key, val in line_in_line_dict.items():
                    if key in line:
                        lines.append(val + '\n')
                        appended = True
                if not appended:
                    lines.append(line)
    
        with open(file, "w") as f:
            f.writelines(lines)
    
    
    # is write permission granted
    def is_file_write_permission_granted(file_path):
        return os.access(file_path, os.W_OK)
    
    
    # is read permission granted
    def is_file_read_permission_granted(file_path):
        return os.access(file_path, os.R_OK)
    
    
    # is file exists
    def is_file_exists(file_path):
        return os.path.exists(file_path)
    
    
    # is directory exists
    def is_dir_exists(dir_path):
        return os.path.isdir(dir_path)
    
    
    # will copy the content of a directory to another directory
    def copy_dir_content(dir_src, dir_dest):
        from distutils.dir_util import copy_tree
        copy_tree(dir_src, dir_dest)
    
    
    # will clear the content of a directory
    def clear_dir_content(dir_path):
        shutil.rmtree(dir_path)
        create_dir(dir_path)
    
    
    # will create a directory
    def create_dir(dir_path):
        os.makedirs(dir_path)

And more...


## Links
[GitHub - osapps](https://github.com/osfunapps)

## Licence
ISC