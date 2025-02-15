#!/usr/bin/env python3
import os
import sys

def main():
    # Prompt for the environment type.
    env_input = input(
        "Select the ENV type:\n"
        "  (A)utomation, (C)I, or (Q)A\n"
        "You can also type full names like 'automation', 'ci', or 'quality assurance': "
    ).strip().lower()
    
    if env_input in ["a", "automation", "auto"]:
        prefix = "qa-automation-"
    elif env_input in ["c", "ci", "continuous integration"]:
        prefix = "ci-graphite-"
    elif env_input in ["q", "qa", "quality assurance"]:
        prefix = "qa-"
    else:
        print("Invalid ENV type selected.")
        sys.exit(1)
    
    # Prompt for the list of numbers.
    numbers_input = input(
        "Enter numbers separated by spaces (from 01 to 36, e.g., 02 03 04): "
    ).strip()
    numbers = numbers_input.split()
    
    # Define your basket of custom commands.
    basket_commands = "kgp; asd; kli"
    
    # For each number, build and run a single shell command.
    for num in numbers:
        full_env = f"{prefix}{num}"
        # The command does:
        #   1. Launch zsh interactively (-i) so that ~/.zshrc is automatically sourced.
        #   2. Explicitly source ~/.zshrc (optional if -i already does it).
        #   3. Connect to the environment with: sk <full_env>
        #   4. Run your basket of commands.
        #   5. Exit the environment with: usk
        command = (
            f"zsh -i -c 'source ~/.zshrc; sk {full_env}; {basket_commands}; usk'"
        )
        print(f"\nRunning in environment: {full_env}")
        # os.system runs the entire sequence in one shell session.
        os.system(command)
    
if __name__ == "__main__":
    main()
