## Repository Testing For Private Fork For Solution and Syncing Updates With Public No Solution Version

My strategy is use private repo homework_and_solution_test as a place to push the document with the newest update homework with the solution. Then we can run the script to detect where the solution begins and ends in each cell and substitute with a comment "## Complete the code below". The homework files with no solution will be stored in separate folder that will be push to the public repo.

# 1. Create Private and Public Repos
I created homework_and_solution_test as a private repo and cleaned_homework_test as a public repo.

# 2. Synchronizing Private to Public Repo
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
# 3. Create a Python Code to strip solutions
On local computer, I wrote the python file for stripping solutions and push it onto private github called stripping_solutions.py The python code is as follow
```
import nbformat
import re
from pathlib import Path

#function for replacing the solution
def strip_solutions(notebook_path, output_path):
    # Load notebook
    nb = nbformat.read(notebook_path, as_version=4)
    
    #check if for the code cell type for a comment where solution begins and ends
    for cell in nb.cells:
        if cell.cell_type == "code":
            new_source = re.sub(
                r"## Begin of Solution.*?## End of Solution",
                "## Complete the code below",
                cell.source,
                flags=re.DOTALL
            )
            cell.source = new_source
    
    # Write out the stripped notebook
    nbformat.write(nb, output_path)
    
#runscript when executed
if __name__ == "__main__":
    solution_dir = Path(".") #current directory
    public_dir = Path("public_notebooks") #make a path for solutions to be stored in public_notebooks
    public_dir.mkdir(exist_ok=True)
    
    #all jupyter notebook in the directory, delete solution from file name and strip out the solution
    for nb_path in solution_dir.glob("*.ipynb"):
        out_path = public_dir / nb_path.name.replace("_solution", "")
        strip_solutions(nb_path, out_path)
        print(f"Created: {out_path}")
```


# 4. Make Edit or Push New Documents onto Private Repo
## Push New Documents
I also make a mock homework with solution where I contain the solution in between comments ## Begin of Solution and ## End of Solution.
Once I am done with working on the homework with solution file, I push the files to the private repo origin/main
```
git pull
git checkout main
git add .
git commit -am "finish HW1 sol and stripping sol file"
git push origin main
```
