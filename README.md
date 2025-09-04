# Repository Testing For Private Fork For Solution and Syncing Updates With Public No Solution Version

My strategy is to use private repo homework_and_solution_test as a place to push the document with the newest update homework with the solution. Then we can run the script to detect where the solutions are in the homework. The homework files with no solution will be stored in separate folder that will be push to the public repo.

## 1. Create Private and Public Repos
I created homework_and_solution_test as a private repo and cleaned_homework_test as a public repo.

## 2. Synchronizing Private to Public Repo
clone the private repo locally
```
git clone https://github.com/Protprommart/homework_and_solution_test.git
```
Go inside private repo
```
cd 'C:\\Users\\Waree\\homework_and_solution_test'
```
use git remote add to make a connection to a public repo
```
git remote add public https://github.com/Protprommart/cleaned_homework_test.git
```
Check for all the remote in private repo, there should be origin (private) and public (public)
```
git remote -v
[output]:
origin  https://github.com/Protprommart/homework_and_solution_test.git (fetch)
origin  https://github.com/Protprommart/homework_and_solution_test.git (push)
public  https://github.com/Protprommart/cleaned_homework_test.git (fetch)
public  https://github.com/Protprommart/cleaned_homework_test.git (push)

```
## 3. Create a Python Code to strip solutions
On local computer, I wrote the python file for stripping solutions and push it onto private github called stripping_solutions.py The python code is as follow
```
import nbformat
import re
from pathlib import Path

#function for replacing the solution
def strip_solutions(notebook_path, output_path):
    # Load notebook
    nb = nbformat.read(notebook_path, as_version=4)
    

    for cell in nb.cells:
        #check if for the code cell type for a comment where solution begins and ends
        if cell.cell_type == "code":
            new_source = re.sub(
                r"## Begin of Solution.*?## End of Solution",
                "## Complete the code below",
                cell.source,
                flags=re.DOTALL
            )
            cell.source = new_source
        #check if the markdown cell has solution as a tag in metadata
        if "solution" in cell.metadata.get("tags",[]):
            cell.source = ("**Answers:** \n ##Insert the answers below")
    
    # Write out the stripped notebook
    nbformat.write(nb, output_path)
    
#runscript when executed
if __name__ == "__main__":
    solution_dir = Path(".").absolute() #current directory
    #find the path to the private repo locally
    private_repo_dir = solution_dir.parent
    public_dir = private_repo_dir / "public_notebooks" #make a path for solutions to be stored in public_notebooks
    public_dir.mkdir(exist_ok=True)
    
    #all jupyter notebook in the directory, delete solution from file name and strip out the solution
    for nb_path in solution_dir.glob("*.ipynb"):
        out_path = public_dir / nb_path.name.replace("_solution", "")
        strip_solutions(nb_path, out_path)
        print(f"Created: {out_path}")
        #print(f"Solution Dir: {solution_dir}, Private Repo Dir: {private_repo_dir}")
```


## 4. Make Edit or Push New Documents onto Private Repo
### Push New Documents
I also made a mock homework with solution where it contains the solution in between comments ## Begin of Solution and ## End of Solution.
Once I am done with working on the homework with solution file, I push the files to the private repo origin/main
```
git pull # making sure local and git hub origin/main are the same
git checkout main
git add .
git commit -am "finish HW1 sol and stripping sol file"
git push origin main
```

## 5. Make a folder to store notebooks with solutions
```
mkdir notebooks_with_solutions
mv hw1_solution.ipynb notebooks_with_solutions/
mv strip_solutions.py notebooks_with_solutions/
```

## 6. Stripping solution of homework 
Inside the notebooks_with_solution folder, create a stripping solution of the homework
```
cd notebooks_with_solutions
python strip_solutions.py

```
From the file hw1_solution, the output is hw1 inside the public_notebooks folder with no solutions in the coding cells
<img width="730" height="589" alt="image" src="https://github.com/user-attachments/assets/0bd13812-2839-4e1c-9cfc-591f8c458933" />
<img width="757" height="456" alt="image" src="https://github.com/user-attachments/assets/4359e639-4d58-4aeb-bcdc-4a59c356c3d6" />

In the case of written answer in the markdown cell, I used a metadata tag in the solution markdown cell called "solution"
<img width="929" height="382" alt="image" src="https://github.com/user-attachments/assets/d614b1ba-81fd-4bb3-ba65-2f476877f83f" />

**Direction**: To achieve metadata tags, in Jupyter Lab or Notebook 
* Select the cell → go to View → Cell Toolbar → Tags.
* Add a tag: solution
After executing the python code, this will be the result
<img width="1003" height="198" alt="image" src="https://github.com/user-attachments/assets/dba9dd11-cec3-416e-b931-d76903008c4c" />

After I ran the python file, the public notebooks would appear in the new folder called public_notebooks
```
[Output:]
Created: C:\Users\Waree\homework_and_solution_test\public_notebooks\hw1.ipynb
```
## 7. Publish only public_notebooks folder to the public repo
make sure to git push all the changes made on local computer to main branch before
Then, make an orphan branch called public_branch and delete all the files from the branch. 
This is to prevent the history of private repo inside the repo.
```
git checkout --orphan public_branch
git rm -rf .
git commit --allow-empty -m "Initial commit for public notebooks"
git push -u origin public_branch
```
copy only public notebooks folder to the public_branch from private main
```
git checkout main -- ../public_notebooks
cp -r ../public_notebooks .
git push -u origin public_branch
```
push the files into the public repo
```
git add .
git commit -m "Add public notebooks only"
git push -u public public_branch:main
```
At this point, the public_branch of the private repo should be the same as public repo
<img width="860" height="489" alt="image" src="https://github.com/user-attachments/assets/1dd337bb-e805-421f-9d21-fad77eb7ba54" />
<img width="863" height="346" alt="image" src="https://github.com/user-attachments/assets/2c43b069-17b6-490b-90f1-719da11bbb84" />


## 8. Summary of What is neccesary for the python file to effectivly remove solutions
1. Coding Solutions need to be wrap by the comment before and after the section attending for students to write
   ```
   ## Begin of Solution
   Code Solution
   ## End of Solution
    
   ```
2. Markdown Solutions cell need to be tag with Solution metadata in jupyter notebook cells
<img width="929" height="382" alt="image" src="https://github.com/user-attachments/assets/d614b1ba-81fd-4bb3-ba65-2f476877f83f" />

3. Name the file in the format of where name of assignment + "solution". In that way, the python script will remove just the solution string from the file name.
