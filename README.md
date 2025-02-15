#!/usr/bin/env python3
import os

def main():
    # Prompt for environment type
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
        return

    # Prompt for numbers (e.g., "02 03 04")
    numbers_input = input("Enter numbers separated by spaces (from 01 to 36, e.g., 02 03 04): ").strip()
    numbers = numbers_input.split()

    # Define the list of custom commands to run.
    custom_commands = ["kgp", "asd", "kli"]

    # For each number, run the custom commands.
    for num in numbers:
        full_env = f"{prefix}{num}"
        print(f"\nsk {full_env}")
        for cmd in custom_commands:
            # Construct the command:
            # - 'zsh -i -c' starts an interactive zsh shell,
            # - 'source ~/.zshrc;' ensures your zsh configuration (and custom commands) are loaded,
            # - then the desired command is executed.
            command = f"zsh -i -c 'source ~/.zshrc; {cmd}'"
            print(f"Running command: {cmd}")
            os.system(command)
        print("usk")

if __name__ == '__main__':
    main()
