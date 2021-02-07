Introduction
------------

this module contains file handling functions to implement in a Python project.

## Installation
Install via pip:

    pip install os-file-handler


## Usage       
Require FileHandler:
```python        
import ostools.file_handler as fh
```

# FileHandler
```python
# will return the content of a directory (full paths)
def get_dir_content(dir_path, recursive=False, collect_dirs=True, collect_files=True, ignored_files_arr=['.DS_Store'], sort_by_name=True):
    f_list = []
    d_list = []
    for path, dirs, files in os.walk(dir_path):
        if collect_dirs:
            if not recursive:
                d_list = append_path_to_list(path, dirs)
                if collect_files:
                    f_list = append_path_to_list(path, files)
                break
            d_list += append_path_to_list(path, dirs)
        if collect_files:
            if not recursive:
                f_list = append_path_to_list(path, files)
                break

            f_list += append_path_to_list(path, files)

    f_list = list(filter(lambda x: get_file_name_from_path(x) not in ignored_files_arr, f_list))
    if f_list:
        if sort_by_name:
            f_list.sort()
        if d_list:
            if sort_by_name:
                d_list.sort()
            return [d_list, f_list]
        return f_list
    return d_list


# will append a base path to every file in a file list
def append_path_to_list(base_path, file_list):
    full_path_file_list = []
    for file in file_list:
        full_path_file_list.append(os.path.join(base_path, file))
    return full_path_file_list


# will return the name of the last dir name from a path
def get_dir_name(dir_path):
    return os.path.basename(os.path.normpath(dir_path))


# will split path to arr of dirs
def split_path(path):
    path = os.path.normpath(path)
    return path.split(os.sep)


# will create a file
def create_file(path, content: list = None):
    with open(path, 'w') as f:
        if content:
            f.writelines(content)


# will create a file from bytes
def bytes_to_file(output_path, data: bytes):
    out_file = open(output_path, "wb")
    out_file.write(data)
    out_file.close()


# will return the extension of a file from a file path
def get_extension_from_file(file):
    _, file_extension = os.path.splitext(file)
    return file_extension


# will return the file name from a file path
def get_file_name_from_path(file, with_extension=True):
    import ntpath
    filename = ntpath.basename(file)
    if not with_extension:
        last_dot_idx = filename.rfind('.')
        filename = filename[:last_dot_idx]
    return filename


# will remove a directory
def remove_dir(path):
    if os.path.isdir(path):
        shutil.rmtree(path)


# will remove a bunch of files in a list
def remove_files(file_list):
    for file in file_list:
        remove_file(file)


def copy_file(src, dst, create_path_if_needed=True):
    """
    Will copy a file to a dest.

    param create_path_if_needed: set to true if the parent directory could not exists and you want to create it
    """
    if not is_dir_exists(get_parent_path(dst)) and create_path_if_needed:
        create_dir(get_parent_path(dst))

    shutil.copy(src, dst)


# will duplicate a file to the same dir with _temp at the end of it's name
def copy_to_temp_file(src):
    file_full_name = get_file_name_from_path(src)
    temp_name = file_full_name[0: file_full_name.rfind('.')] + '_temp'
    extension = get_extension_from_file(file_full_name)
    temp_dest = get_parent_path(src) + '/' + temp_name + extension
    copy_file(src, temp_dest)
    return temp_dest


# will return the parent path of a file
def get_parent_path(file):
    from pathlib import Path
    return str(Path(file).parent)


# will copy a list of files to a dir
def copy_list_of_files(files_list, dst):
    for file in files_list:
        copy_file(file, dst, create_path_if_needed=True)


# will rename a file or a directory
def rename(src, dst):
    os.rename(src, dst)


# will search for a file in a path by prefix, suffix, full name with/without an extension
def search_file(path_to_search, full_name=None, prefix=None, suffix=None, by_extension=None, recursive=True):
    from pathlib import Path
    files = []
    search_queue = ''
    if recursive:
        search_queue = '**/'
    if full_name:
        search_queue += full_name
        if by_extension:
            search_queue += by_extension
    else:
        if prefix:
            search_queue += f'{prefix}'
            if suffix:
                search_queue += f'*{suffix}'
            else:
                if not by_extension:
                    search_queue += '*'
        else:
            if suffix:
                search_queue += f'*{suffix}'
        if by_extension:
            search_queue += f'*{by_extension}'
        else:
            search_queue += f'.*'

    for filename in Path(path_to_search).glob(search_queue):
        files.append(str(filename))
    return files


# will search for a directory in a path by full name or prefix or/and suffix
def search_dir(path_to_search, full_name=None, prefix=None, suffix=None, recursive=True):
    from pathlib import Path
    files = []
    search_queue = ''
    if recursive:
        search_queue = '**/'

    if full_name:
        search_queue += full_name
    elif prefix:
        search_queue += f'{prefix}'
        if suffix:
            search_queue += f'*{suffix}'
        else:
            search_queue += '*'
    elif suffix:
        search_queue += f'*{suffix}'

    for filename in Path(path_to_search).glob(search_queue):
        if is_dir_exists(filename):
            files.append(str(filename))
    return files


def replace_line_for_line(file, line_for_line_dict):
    """
    Will replace a line for a line in a file.
    Notice: this function use contains() and not ==.

    Args:
        param file:
        param line_for_line_dict:
    """
    lines = []
    with open(file, "r") as f:
        for line in f:
            appended = False
            for key, val in line_for_line_dict.items():
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


def is_dir_empty(dir_path):
    return len(os.listdir(dir_path)) == 0


# will copy a directory
def copy_dir(src, dst, ignore_patterns_str='.DS_Store', overwrite_content_if_exists=False):
    from shutil import copytree, ignore_patterns
    if is_dir_exists(dst):
        for idx_dir in get_dir_content(src, recursive=True, collect_dirs=True, collect_files=False):
            dst_dir_paths = idx_dir.replace(f'{src}/', '')
            dir_dst = os.path.join(dst, dst_dir_paths)
            copy_dir(idx_dir, dir_dst, overwrite_content_if_exists=overwrite_content_if_exists)

        for idx_file in get_dir_content(src, recursive=True, collect_dirs=False, collect_files=True):
            f_path = dst_dir_paths = idx_file.replace(f'{src}/', '')
            file_dst = os.path.join(dst, f_path)
            copy_file(idx_file, file_dst, create_path_if_needed=True)
    else:
        copytree(src, dst, ignore=ignore_patterns(ignore_patterns_str))


# will clear the content of a directory
def clear_dir_content(dir_path):
    shutil.rmtree(dir_path)
    create_dir(dir_path)


# will create a directory. If dir exists, do nothing
def create_dir(dir_path):
    if not is_dir_exists(dir_path):
        os.makedirs(dir_path)


# will remove all of the files with a given extension
def remove_all_files_with_extension(path, ext):
    import glob
    if ext[0] == '.':
        ext = ext[1:]
    for f in glob.glob(path + "/*." + ext):
        remove_file(f)


# will remove a single file
def remove_file(file):
    if is_file_exists(file):
        os.remove(file)


# will check if line exists in a file
def is_line_exists_in_file(file, line_to_find):
    with open(file) as f:
        content = f.readlines()
        for line in content:
            if line_to_find in line:
                return True
    return False


# will read a file and look for a string. If the string exists, will return the whole line which comprise it
def get_line_from_file(file, str_to_find):
    with open(file) as read_file:
        for line in read_file:
            if str_to_find in line:
                return line
    return None


# will turn a json file to a dictionary
def json_file_to_dict(json_file):
    import json

    with open(json_file) as f:
        data = json.load(f)
    return data


# will turn a dictionary to a json file
def dict_to_json_file(json_file, dictt):
    import json
    with open(json_file, 'w') as f:
        json.dump(dictt, f)


def remove_lines_from_file(file_path, lines_arr_to_remove=None, remove_from=None, remove_until=None):
    """
    Will remove lines from a file if the lines contains certain strings
    Args:
        param file_path: the path to your file
        param lines_arr_to_remove: (optional) an array of lines to search for (this function calls contains() and not == )
        param remove_from: (optional) if you want to remove a range of lines, set here the first line in which to start the removal)
        param remove_until: (optional) if you want to remove a range of lines, set here the last line in which to end the removal)
    """
    if lines_arr_to_remove is None:
        lines_arr_to_remove = []
    import fileinput
    import sys
    on_remove_sequence = False
    for line in fileinput.input(file_path, inplace=1):
        line_to_remove_found = False
        if remove_from is not None and remove_from in line:
            on_remove_sequence = True

        if remove_until is not None and remove_until in line:
            on_remove_sequence = False
            continue

        if on_remove_sequence:
            continue

        if lines_arr_to_remove is not None:
            for line_to_remove in lines_arr_to_remove:
                if line_to_remove in line:
                    line_to_remove_found = True
                    break

        if line_to_remove_found:
            continue

        sys.stdout.write(line)


def file_to_bytes(path, n=None):
    """
    Will read a file to bytes
    Args:
      param path: the path to your file
      param n: the bytes count to read (leave None to read the whole file)
    """
    in_file = open(path, "rb")
    data = in_file.read(n)
    in_file.close()
    return data


# will extract a zip file to a destination
def extract_zip_file(zip_file_path, dst_path):
    from zipfile import ZipFile
    # Create a ZipFile Object and load sample.zip in it

    with ZipFile(zip_file_path, 'r') as zipObj:
        # Extract all the contents of zip file in current directory
        zipObj.extractall(dst_path)
```

And more...

## Links
[GitHub - osapps](https://github.com/osfunapps)

## Licence
ISC