#!/usr/bin/env python3
import os
import sys
import logging

# Define a custom formatter that adds colors to console log messages.
class ColoredFormatter(logging.Formatter):
    COLORS = {
        'DEBUG': '\033[36m',    # Cyan
        'INFO': '\033[32m',     # Green
        'WARNING': '\033[33m',  # Yellow
        'ERROR': '\033[31m',    # Red
        'CRITICAL': '\033[41m'  # Red background
    }
    RESET = '\033[0m'
    
    def format(self, record):
        color = self.COLORS.get(record.levelname, self.RESET)
        message = super().format(record)
        return f"{color}{message}{self.RESET}"

def setup_logging():
    """Configure logging with colored output on the console and plain output to a log file."""
    logger = logging.getLogger()
    logger.setLevel(logging.DEBUG)
    
    # Create console handler with colored formatter.
    ch = logging.StreamHandler(sys.stdout)
    ch.setLevel(logging.DEBUG)
    ch_formatter = ColoredFormatter('%(asctime)s %(levelname)s: %(message)s', datefmt='%Y-%m-%d %H:%M:%S')
    ch.setFormatter(ch_formatter)
    logger.addHandler(ch)
    
    # Create file handler with plain formatter.
    fh = logging.FileHandler("env_script.log")
    fh.setLevel(logging.DEBUG)
    fh_formatter = logging.Formatter('%(asctime)s %(levelname)s: %(message)s', datefmt='%Y-%m-%d %H:%M:%S')
    fh.setFormatter(fh_formatter)
    logger.addHandler(fh)

def main():
    setup_logging()
    logging.info("=== Script started ===")
    
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
        logging.error("Invalid ENV type selected.")
        sys.exit(1)
    
    # Prompt for the list of numbers.
    numbers_input = input("Enter numbers separated by spaces (from 01 to 36, e.g., 02 03 04): ").strip()
    numbers = numbers_input.split()
    
    # Define the basket of custom commands.
    basket_commands = "kgp; asd; kli"
    
    # For each number, build and run a single shell command.
    for num in numbers:
        full_env = f"{prefix}{num}"
        logging.info(f"Running in environment: {full_env}")
        # Build the full command. The -i flag ensures zsh runs interactively and sources ~/.zshrc.
        # The sequence is: connect to environment (sk <env>), run the basket, then exit (usk)
        command = f"zsh -i -c 'source ~/.zshrc; sk {full_env}; {basket_commands}; usk'"
        logging.debug(f"Full command: {command}")
        os.system(command)
    
    logging.info("=== Script ended ===")

if __name__ == "__main__":
    main()
