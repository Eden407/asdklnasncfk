#!/usr/bin/env python3
import subprocess
import logging
import os
import sys
import select
import time

def setup_logging():
    """Configure logging to both console and a log file."""
    logging.basicConfig(
        level=logging.DEBUG,  # Set to DEBUG to see more details.
        format='%(asctime)s %(levelname)s: %(message)s',
        handlers=[
            logging.StreamHandler(sys.stdout),
            logging.FileHandler("env_script.log")
        ]
    )

def run_command_in_session(shell, command, timeout=5):
    """
    Send a command to the persistent shell session and capture its output.
    
    This function writes the command to shell.stdin, appending:
      - an echo of the exit code (prefixed with __EXIT_CODE__)
      - an echo of a marker (__COMMAND_END__)
    
    It then uses select to wait for output until the marker is encountered
    or the timeout is reached.
    
    Returns a tuple (output, exit_code).
    """
    marker = "__COMMAND_END__"
    exit_marker_prefix = "__EXIT_CODE__"
    
    # Send the command along with marker instructions.
    shell.stdin.write(command + "\n")
    shell.stdin.write(f"echo {exit_marker_prefix}$?\n")
    shell.stdin.write(f"echo {marker}\n")
    shell.stdin.flush()
    
    output_lines = []
    exit_code = None
    start_time = time.time()
    
    while True:
        # Wait for output for up to 'timeout' seconds.
        rlist, _, _ = select.select([shell.stdout], [], [], timeout)
        if rlist:
            line = shell.stdout.readline()
            if not line:
                break  # End-of-file.
            line = line.rstrip("\n")
            # Look for our exit code marker.
            if line.startswith(exit_marker_prefix):
                try:
                    exit_code = int(line[len(exit_marker_prefix):])
                except ValueError:
                    exit_code = -1
            elif line == marker:
                break  # End marker reached.
            else:
                output_lines.append(line)
        else:
            logging.warning("Timeout reached while waiting for output from command: %s", command)
            break  # Timeout reached.
        # (Optional) Uncomment to debug elapsed time:
        # logging.debug("Elapsed time: %.2f", time.time()-start_time)
    return "\n".join(output_lines), exit_code

def main():
    setup_logging()
    logging.info("=== Script started ===")
    
    # Start a persistent interactive zsh session.
    shell = subprocess.Popen(
        ["/bin/zsh", "-i"],
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,
        text=True,
        bufsize=1,
    )
    
    # Clear the prompt (set PS1 to empty) so prompt text does not mix with our markers.
    shell.s
