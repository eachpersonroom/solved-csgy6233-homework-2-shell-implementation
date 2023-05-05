Download Link: https://assignmentchef.com/product/solved-csgy6233-homework-2-shell-implementation
<br>
In this assignment, you will implement pieces of the UNIX shell and get some familiarity with a few UNIX library calls along with the UNIX process model. By the end of the assignment, you will have

implemented a shell that can run any series of complex pipelines of commands, such as:




$ cat words.txt | grep cat | sed s/cat/dog/ &gt; doggerel.txt




The pipeline shown above takes a word document labelled words.txt (a file generally installed on UNIX systems that contains a list of English words.  This file will be included to download on Brightspace), select the words containing the string “cat”, and then uses sed to replace “cat” with “dog”, so that, for example, “concatenate” becomes “condogenate”. The results are then outputted to a text file labelled “doggerel.txt”. (You can find detailed descriptions of each of the commands in the pipeline by consulting the manual page for the command; e.g.: “man grep” or “man sed”.)







Start by downloading the shell.c and words.txt.  Shell.c is a skeleton file attached to this homework which you will place in your Anubis IDE. You don’t have to understand how the parser works in detail, but you should have a general idea of how the flow of control works in the program. You will also see comments labelled with “//your code here”, which is where you will implement the functionality to make the shell actually work. Next, you must try to compile the source

code to the Anubis Unix shell (Again, this will not be using xv6):




$ gcc shell.c -o shell




You can then run and interact with the shell by typing ./shell :




<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="c0b5b3a5b280a3b3f6f2f3f3">[email protected]</a>:~$ ./shell




cs6233&gt; ls




exec not implemented




cs6233&gt;




Note: The command prompt for our shell is set to cs6233&gt; to make it easy to tell the difference between our shell and the Anubis’s Linux shell. You can quit your shell by typing Ctrl-C or Ctrl-D




Problem 1: Command Execution




Implement basic command execution by filling in the code inside of the case’ ‘ blocking the runcmd




function. You will want to look at the manual page for the exec(3) function by typing “man 3 exec” (Note: throughout this course, when referring to commands that one can look up in the man pages, we will typically specify the section number in parentheses — thus, since exec is found in section 3, we will say exec(3)).




Once this is done, you should be able to use your shell to run single commands, such as




cs6233&gt; ls




cs6233&gt; grep cat




words.txt




Hint:




You will notice that there are many variants on exec(3). You should read through the differences between them in the xv6 manual, and then choose the one that allows you to run the commands above — in particular, pay attention to whether the version of exec you’re using requires you to enter in the full path to the program, or whether it will search the directories in the PATH environment variable.







Problem 2: I/O Redirection







Now extend the shell to handle input and output redirection. Programs will be expecting their input on standard input and write to standard output, so you will have to open the file and then replace standard input or output with that file. As before, the parser already recognizes the ‘&gt;’ and ‘&lt;‘ characters and builds a redircmd structure for you, so you just need to use the information in that redircmd to open a file and replace standard input or output with it.




Hints:




<ol>

 <li>Look at the dup2(2) and open(2) calls.</li>

 <li>The file descriptor the program is currently using for input or output is available in rcmd-&gt;fd. If you’re confused about where rcmd-&gt;fd is coming from, look at the redircmd function and remember that 0 is standard input, 1 is standard output.</li>

 <li>Be careful with the open call; in particular, make sure you read about the case when you pass the O_CREAT flag.</li>

</ol>







When this is done, you should be able to redirect the input and output of commands:




cs6233&gt; ls &gt; a.txt




cs6233&gt; sort -r &lt; a.txt










Make sure to explain your program or approach in a few sentences and submit your explanation in a text document on Brightspace










Problem 3: Pipes







The final task is to add the ability to pipe the output of one command into the input of another.




You will fill out the code for the ‘|’ case of the switch statement in runcmd to do this.







Hints:




<ol>

 <li>The parser provides the left command in pcmd-&gt;left and the right command in pcmd- &gt;right.</li>

</ol>




<ol start="2">

 <li>Look at the fork(2), pipe(2), close(2) and wait(2) calls.</li>

</ol>




<ol start="3">

 <li>If your program just hangs, it may help to know that reads to pipes with no data will block until all</li>

</ol>

file descriptors referencing the pipe are closed.




<ol start="4">

 <li>Note that fork(2) creates an exact copy of the current process. The two processes share any</li>

</ol>

file descriptors that were open at the time the fork occurred. You can get a sense for this:




#include &lt;stdio.h&gt;




#include &lt;unistd.h&gt;




#include &lt;fcntl.h&gt;




#include &lt;sys/stat.h&gt;




int main() {




int filedes;




filedes = open(“myfile.txt”, O_RDWR | O_CREAT, S_IRUSR | S_IWUSR); int rv;

rv = fork();




if (rv == 0) {




char msg[] = “Process 1
”;




printf(“Hello, I’m in the child, my process ID is %d
”,getpid());

write(filedes, msg, sizeof(msg));




}




else {




char msg[] = “Process 2
”;




printf(“This is the parent process, my process ID is %d and my child is %d
”, getpid(), rv);

write(filedes, msg, sizeof(msg));




}







close(filedes);




}







If you put that code into a separate file in Anubis, compile it, and then run the resulting program, you should see a result like:




<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="e889869d8a819ba889869d8a819bc5818c8d">[email protected]</a>:~/homework2_fall2021-cs6233student$:~$ gcc a.c -o a.out

<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="a2c3ccd7c0cbd1e2c3ccd7c0cbd18fcbc6c7">[email protected]</a>:~/homework2_fall2021-cs6233student$:~$ ./a.out




This is the parent process, my process ID is [Random Number X] and my child is [Random Number X + 1]

Hello, I’m in the child, my process ID is [Random Number X + 1]




<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="e0818e95828993a0818e95828993cd898485">[email protected]</a>:~/homework2_fall2021-cs6233student$:~$ cat myfile.txt




Process 2

Process 1




You can see that both the parent and child process both got a copy of “filedes”, and that writes to it from each process went to the same underlying file.




<ol start="5">

 <li>You may find it helpful to re-read the first chapter of the xv6 manual, which describes in detail how the xv6 shell works. Note that the code shown there will not work as-is — you will have to adapt it for the Anubis IDE Unix environment.</li>

</ol>

Once this is done, you should be able to run a full pipeline:




cs6233&gt; cat words.txt | grep cat | sed s/cat/dog/ &gt;




doggerel.txt




cs6233&gt; grep con &lt; doggerel.txt




Explain your program or approach in a few sentences (10 points)

<h3>Explain in detail: –</h3>




Q1. How would you implement lists of commands, separated by “;” –







<table width="548">

 <tbody>

  <tr>

   <td colspan="2" width="548">Q3. How would you implement running commands in the background by</td>

  </tr>

  <tr>

   <td width="301">supporting “&amp;” and “wait” –</td>

   <td width="248"></td>

  </tr>

 </tbody>

</table>







     You may submit your modified shell.c in Anubis.  In addition, you must also submit a pdf/word file to the Brightspace assignment containing your written answers.


