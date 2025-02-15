#!/usr/bin/env python3
import subprocess
import logging
import os

def setup_logging():
    """Configure logging to both console and file."""
    logging.basicConfig(
        level=logging.INFO,
        format='%(asctime)s %(levelname)s: %(message)s',
        handlers=[
            logging.FileHandler("env_script.log"),
            logging.StreamHandler()
        ]
    )

def run_command_in_session(shell, command):
    """
    Run a command in the persistent zsh shell session.
    
    Writes the command to shell.stdin, then appends:
      - an echo of the exit code (prefixed with __EXIT_CODE__)
      - an echo of a marker (__COMMAND_END__)
    
    Reads from shell.stdout until the marker is encountered.
    Returns a tuple (output, exit_code).
    """
    marker = "__COMMAND_END__"
    exit_marker_prefix = "__EXIT_CODE__"
    
    # Send the command to the shell.
    shell.stdin.write(command + "\n")
    shell.stdin.write(f"echo {exit_marker_prefix}$?\n")
    shell.stdin.write(f"echo {marker}\n")
    shell.stdin.flush()
    
    output_lines = []
    exit_code = None
    
    # Read until the marker is reached.
    while True:
        line = shell.stdout.readline()
        if not line:
            break  # End-of-file.
        stripped_line = line.rstrip("\n")
        if stripped_line.startswith(exit_marker_prefix):
            try:
                exit_code = int(stripped_line[len(exit_marker_prefix):])
            except ValueError:
                exit_code = -1
        elif stripped_line == marker:
            break
        else:
            output_lines.append(stripped_line)
    
    return "\n".join(output_lines), exit_code

def main():
    setup_logging()
    logging.info("=== Script started ===")
    
    # Start a persistent interactive zsh shell session.
    # The '-i' flag ensures that your ~/.zshrc is automatically sourced.
    shell = subprocess.Popen(
        ["/bin/zsh", "-i"],
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,
        text=True,
        bufsize=1,
    )
    
    # Prompt for environment type with flexible input.
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
        print("Invalid option. Please choose a valid ENV type (A, C, or Q).")
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
    
    # List of custom commands to run.
    custom_commands = ["kgp", "asd", "kli"]
    
    # Process each number.
    for num in numbers:
        full_env = f"{prefix}{num}"
        print(f"\nsk {full_env}")
        logging.info(f"sk {full_env}")
        
        # Execute each custom command.
        for cmd in custom_commands:
            logging.info(f"Executing command: {cmd}")
            output, code = run_command_in_session(shell, cmd)
            
            # Print output to the console.
            print(f"\nOutput from '{cmd}':")
            print(output)
            print(f"Exit code: {code}")
            
            # Log output and status.
            logging.info(f"Output from '{cmd}':\n{output}")
            if code is None or code != 0:
                logging.error(f"Command '{cmd}' failed with exit code {code}")
            else:
                logging.info(f"Command '{cmd}' completed successfully with exit code {code}")
        
        print("usk")
        logging.info("usk")
    
    logging.info("=== Script ended ===")
    
    # Terminate the persistent zsh shell session.
    shell.stdin.write("exit\n")
    shell.stdin.flush()
    shell.wait()

if __name__ == "__main__":
    main()
