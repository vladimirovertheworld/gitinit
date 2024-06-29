# GitHub Repository Initialization Script

This script helps initialize a Git repository, configure a remote origin, create an initial commit, and push the changes to a GitHub repository. It also handles SSH key generation and conflict resolution.

## Features

- Initializes a new Git repository
- Checks for SSH keys and generates them if necessary
- Adds a remote origin
- Creates an initial commit
- Pushes changes to the remote repository
- Handles merge conflicts during push

## Prerequisites

- Ensure Git is installed on your system
- Ensure you have an SSH key added to your GitHub account (the script can generate one if not found)

## Supported Operating Systems

- Linux (Ubuntu, Fedora, Debian, CentOS, etc.)
- macOS
- Windows (with Git Bash or WSL)

## Usage

1. Save the script as `github_init.sh`.
2. Make the script executable:
   ```bash
   chmod +x github_init.sh
   ```
3. Run the script:
   ```bash
   ./github_init.sh
   ```

## Script Details

### Functions

- **command_exists**: Checks if a command exists on the system.
- **handle_error**: Displays an error message and exits the script.
- **handle_warning**: Displays a warning message.
- **prompt_user**: Prompts the user for a yes/no response.
- **check_ssh_key**: Checks for the existence of an SSH key and generates one if not found.

### Steps

1. **Check if Git is installed**:
   ```bash
   if ! command_exists git; then
       handle_error "git is not installed. Please install git and try again."
   fi
   ```

2. **Check for SSH key**:
   ```bash
   check_ssh_key
   ```

3. **Reinitialize Git repository if needed**:
   ```bash
   if [ -d .git ]; then
       if prompt_user "This directory is already a git repository. Do you want to reinitialize it?"; then
           rm -rf .git || handle_error "Failed to remove existing .git directory"
       else
           echo "Exiting without changes."
           exit 0
       fi
   fi
   ```

4. **Initialize Git repository**:
   ```bash
   git init || handle_error "Failed to initialize git repository"
   echo -e "${GREEN}Git repository initialized successfully.${NC}"
   ```

5. **Add remote origin**:
   ```bash
   while true; do
       read -p "Enter your GitHub repository URL (SSH or HTTPS): " repo_url
       if [[ $repo_url =~ ^(https://github.com/.+/.+\.git|git@github.com:.+/.+\.git)$ ]]; then
           break
       else
           handle_warning "Invalid GitHub repository URL. It should be in the format:"
           echo "HTTPS: https://github.com/username/repository.git"
           echo "SSH: git@github.com:username/repository.git"
       fi
   done

   git remote add origin $repo_url || handle_error "Failed to add remote origin"
   echo -e "${GREEN}Remote origin added successfully.${NC}"
   ```

6. **Create initial commit**:
   ```bash
   git add . || handle_warning "Failed to stage files"
   git commit -m "Initial commit" || {
       handle_warning "Failed to create initial commit. Attempting to configure git user..."
       read -p "Enter your Git username: " git_username
       read -p "Enter your Git email: " git_email
       git config user.name "$git_username" || handle_error "Failed to set git username"
       git config user.email "$git_email" || handle_error "Failed to set git email"
       git commit -m "Initial commit" || handle_error "Failed to create initial commit after configuring user"
   }
   echo -e "${GREEN}Initial commit created successfully.${NC}"
   ```

7. **Push to remote repository and handle conflicts**:
   ```bash
   echo -e "${BLUE}Attempting to push to remote repository...${NC}"
   if ! git push -u origin main; then
       handle_warning "Push to 'main' branch failed. Attempting to pull and resolve conflicts..."
       if git pull origin main --rebase; then
           handle_warning "Conflicts detected. Please resolve them manually."
           echo -e "${BLUE}Follow these steps to resolve conflicts:${NC}"
           echo "1. Open the conflicted files and resolve the conflicts manually."
           echo "2. Use 'git add <conflicted_files>' to mark them as resolved."
           echo "3. Continue the rebase with 'git rebase --continue'."
           echo "4. After resolving conflicts, push the changes with 'git push -u origin main'."
           exit 1
       else
           handle_error "Failed to pull remote changes"
       fi
   else
       echo -e "${GREEN}Successfully pushed to 'main' branch.${NC}"
   fi
   ```

8. **Final checks**:
   ```bash
   if ! git remote -v | grep -q origin; then
       handle_warning "Remote 'origin' is not set. Please check your repository configuration."
   fi

   if [ -z "$(git log -1 --pretty=%B)" ]; then
       handle_warning "No commits found. Ensure you have committed your changes."
   fi

   echo -e "${BLUE}Deployment script execution completed.${NC}"
   ```

## Notes

- Ensure you have the necessary permissions to push to the specified GitHub repository.
- Resolve any merge conflicts as instructed in the script output.

## License

This script is provided as-is without any warranty. Use it at your own risk.
