#!/usr/bin/env python3
import subprocess
import logging
import os
import sys

def setup_logging():
    """Configure logging to both console and a log file."""
    logging.basicConfig(
        level=logging.INFO,
        format='%(asctime)s %(levelname)s: %(message)s',
        handlers=[
            logging.StreamHandler(sys.stdout),
            logging.FileHandler("env_script.log")
        ]
    )

def run_command_in_session(shell, command):
    """
    Send a command to the persistent shell session and capture its output.
    
    The function writes the command to shell.stdin, then appends:
      - an echo of the exit code (prefixed with __EXIT_CODE__)
      - an echo of a marker (__COMMAND_END__)
    
    It then reads from shell.stdout until the marker is encountered.
    Returns a tuple (output, exit_code).
    """
    marker = "__COMMAND_END__"
    exit_marker_prefix = "__EXIT_CODE__"
    
    # Send the command and the marker lines.
    shell.stdin.write(command + "\n")
    shell.stdin.write(f"echo {exit_marker_prefix}$?\n")
    shell.stdin.write(f"echo {marker}\n")
    shell.stdin.flush()
    
    output_lines = []
    exit_code = None
    
    while True:
        line = shell.stdout.readline()
        if not line:
            break  # End-of-file.
        line = line.rstrip("\n")
        if line.startswith(exit_marker_prefix):
            try:
                exit_code = int(line[len(exit_marker_prefix):])
            except ValueError:
                exit_code = -1
        elif line == marker:
            break
        else:
            output_lines.append(line)
    return "\n".join(output_lines), exit_code

def main():
    setup_logging()
    logging.info("=== Script started ===")
    
    # Start a persistent interactive zsh session.
    # The '-i' flag ensures that your ~/.zshrc is sourced.
    shell = subprocess.Popen(
        ["/bin/zsh", "-i"],
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,
        text=True,
        bufsize=1,
    )
    
    # Optionally, force-source your zsh configuration (if not already sourced).
    zshrc_path = os.path.expanduser("~/.zshrc")
    if os.path.exists(zshrc_path):
        out, code = run_command_in_session(shell, f"source {zshrc_path}")
        logging.info(f"Sourced {zshrc_path} (exit code {code}).")
    else:
        logging.warning(f"{zshrc_path} not found. Continuing without sourcing it.")
    
    # Prompt for the environment type.
    env_input = input(
        "Select the ENV type:\n"
        "  (A)utomation, (C)I, or (Q)A\n"
        "You can also type full names like 'automation', 'ci', or 'quality assurance': "
    ).strip().lower()
    
    if env_input in ["a", "automation", "auto"]:
        prefix = "qa-automation-"
        env_name = "Automation"
    elif env_input in ["c", "ci", "continuous integration"]:
        prefix = "ci-graphite-"
        env_name = "CI"
    elif env_input in ["q", "qa", "quality assurance"]:
        prefix = "qa-"
        env_name = "QA"
    else:
        print("Invalid ENV type selected.")
        logging.error("Invalid ENV type selected.")
        shell.stdin.write("exit\n")
        shell.stdin.flush()
        shell.wait()
        return
    
    logging.info(f"Selected ENV: {env_name} with prefix: {prefix}")
    
    # Prompt for numbers (e.g., "02 03 04").
    numbers_input = input("Enter numbers separated by spaces (from 01 to 36, e.g., 02 03 04): ").strip()
    numbers = numbers_input.split()
    logging.info(f"Numbers received: {numbers}")
    
    # Define your basket of custom commands.
    custom_commands = ["kgp", "asd", "kli"]
    
    # For each number, connect to the environment, run the basket, and exit the environment.
    for num in numbers:
        full_env = f"{prefix}{num}"
        
        # 1. Connect to the environment.
        connect_cmd = f"sk {full_env}"
        logging.info(f"Connecting to environment with: {connect_cmd}")
        out, code = run_command_in_session(shell, connect_cmd)
        print(f"\nCommand: {connect_cmd}\nOutput:\n{out}\nExit code: {code}\n")
        
        # 2. Run each custom command within this environment.
        for cmd in custom_commands:
            logging.info(f"Running command '{cmd}' in environment {full_env}")
            out, code = run_command_in_session(shell, cmd)
            print(f"\nCommand: {cmd}\nOutput:\n{out}\nExit code: {code}\n")
        
        # 3. Exit the environment.
        exit_cmd = "usk"
        logging.info(f"Exiting environment with: {exit_cmd}")
        out, code = run_command_in_session(shell, exit_cmd)
        print(f"\nCommand: {exit_cmd}\nOutput:\n{out}\nExit code: {code}\n")
    
    logging.info("=== Script ended ===")
    
    # Terminate the persistent shell session.
    shell.stdin.write("exit\n")
    shell.stdin.flush()
    shell.wait()

if __name__ == "__main__":
    main()
