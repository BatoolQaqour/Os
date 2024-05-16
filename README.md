# Os
Os Project
const fs = require('fs');
const readline = require('readline');

class Process {
    constructor(pid, arrivalTime, burstTime) {
        this.pid = pid;
        this.arrivalTime = arrivalTime;
        this.burstTime = burstTime;
        this.remainingTime = burstTime;
        this.startTime = 0;
        this.finishTime = 0;
        this.waitingTime = 0;
        this.turnaroundTime = 0;
        this.executed = false;
    }
}

function readProcessesFromFile(filePath) {
    const processes = [];
    const lines = fs.readFileSync(filePath, 'utf-8').split('\n');
    for (const line of lines) {
        if (line.trim()) {
            const [pid, arrivalTime, burstTime] = line.trim().split(' ').map(Number);
            if (!isNaN(pid) && !isNaN(arrivalTime) && !isNaN(burstTime)) {
                processes.push(new Process(pid, arrivalTime, burstTime));
            }
        }
    }
    return processes;
}

function fcfsScheduler(processes) {
    let currentTime = 0;
    for (const process of processes) {
        if (currentTime < process.arrivalTime) {
            currentTime = process.arrivalTime;
        }
        process.startTime = currentTime;
        process.finishTime = currentTime + process.burstTime;
        process.turnaroundTime = process.finishTime - process.arrivalTime;
        process.waitingTime = process.turnaroundTime - process.burstTime;
        currentTime = process.finishTime;
    }
}


function sjfScheduler(processes) {
    let currentTime = 0;
    const readyQueue = [];
    const finishedProcesses = [];
    processes.sort((a, b) => a.arrivalTime - b.arrivalTime);  

    while (finishedProcesses.length < processes.length) {
        processes.forEach(process => {
            if (!process.executed && process.arrivalTime <= currentTime) {
                readyQueue.push(process);
                process.executed = true;  
            }
        });

        readyQueue.sort((a, b) => a.burstTime - b.burstTime);

        if (readyQueue.length > 0) {
            const process = readyQueue.shift();
            if (currentTime < process.arrivalTime) {
                currentTime = process.arrivalTime; 
            }
            process.startTime = currentTime;
            process.finishTime = currentTime + process.burstTime;
            process.turnaroundTime = process.finishTime - process.arrivalTime;
            process.waitingTime = process.startTime - process.arrivalTime;
            currentTime = process.finishTime;
            finishedProcesses.push(process);
        } else {
            const nextProcess = processes.find(p => !p.executed);
            if (nextProcess) {
                currentTime = nextProcess.arrivalTime;
            }
        }
    }
    return finishedProcesses;
}


function roundRobinScheduler(processes, quantum) {
    let currentTime = 0;
    const readyQueue = [...processes];
    const finishedProcesses = [];

    while (readyQueue.length > 0) {
        const process = readyQueue.shift();
        if (process.remainingTime > quantum) {
            currentTime += quantum;
            process.remainingTime -= quantum;
            readyQueue.push(process);
        } else {
            currentTime += process.remainingTime;
            process.remainingTime = 0;
            process.finishTime = currentTime;
            process.turnaroundTime = process.finishTime - process.arrivalTime;
            process.waitingTime = process.turnaroundTime - process.burstTime;
            finishedProcesses.push(process);
        }
    }
    return finishedProcesses;
}


function displayResults(processes) {
    let totalWaitingTime = 0;
    let totalTurnaroundTime = 0;
    const totalCpuTime = processes.reduce((sum, p) => sum + p.burstTime, 0);

    console.log('Process ID\tFinish Time\tWaiting Time\tTurnaround Time');
    for (const process of processes) {
        console.log(`${process.pid}\t\t${process.finishTime}\t\t${process.waitingTime}\t\t${process.turnaroundTime}`);
        totalWaitingTime += process.waitingTime;
        totalTurnaroundTime += process.turnaroundTime;
    }

    const cpuUtilization = (totalCpuTime / processes[processes.length - 1].finishTime) * 100;
    console.log(`\nTotal CPU Utilization: ${cpuUtilization.toFixed(2)}%`);
    console.log(`Average Waiting Time: ${(totalWaitingTime / processes.length).toFixed(2)}`);
    console.log(`Average Turnaround Time: ${(totalTurnaroundTime / processes.length).toFixed(2)}`);
}

function main() {
    const filePath = 'C:\\Users\\HP\\Desktop\\test.txt';
    const processes = readProcessesFromFile(filePath);

    const rl = readline.createInterface({
        input: process.stdin,
        output: process.stdout
    });

    console.log("Choose a scheduling algorithm:");
    console.log("1. FCFS (First-Come, First-Served)");
    console.log("2. SJF (Shortest Job First)");
    console.log("3. Round Robin");

    rl.question("Enter the number of the algorithm you want to run: ", (answer) => {
        switch (answer) {
            case '1':
                const fcfsProcesses = [...processes];
                fcfsScheduler(fcfsProcesses);
                console.log('\nFCFS Scheduling Results:');
                displayResults(fcfsProcesses);
                rl.close();
                break;

            case '2':
                const sjfProcesses = [...processes];
                const sjfFinishedProcesses = sjfScheduler(sjfProcesses);
                console.log('\nSJF Scheduling Results:');
                displayResults(sjfFinishedProcesses);
                rl.close();
                break;

            case '3':
                rl.question("Enter the quantum time (e.g., 4): ", (quantumStr) => {
                    const quantum = parseInt(quantumStr, 10);
                    const rrProcesses = [...processes];
                    const rrFinishedProcesses = roundRobinScheduler(rrProcesses, quantum);
                    console.log(`\nRound Robin Scheduling Results (Quantum = ${quantum}):`);
                    displayResults(rrFinishedProcesses);
                    rl.close();
                });
                break;

            default:
                console.log("Invalid choice. Please choose a valid option (1, 2, or 3).");
                rl.close();
                break;
        }
    });
}

main();
