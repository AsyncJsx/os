# BTCOL406: Operating Systems Laboratory

<div style="page-break-after: always;"></div>

## Practical 1: Hands on Unix Commands

```bash
pwd
mkdir myfolder
cd myfolder
touch file1.txt
echo "Hello Linux" > file1.txt
cat file1.txt
cp file1.txt file2.txt
mv file2.txt newfile.txt
ls -l
rm newfile.txt
cd ..
rm -r myfolder
```

<div style="page-break-after: always;"></div>

## Practical 2: Shell Programming for File Handling

```bash
#!/bin/bash
f=test.txt
touch $f
echo "Hello World" > $f
echo "Second line" >> $f
cat $f
mv $f newtest.txt
mkdir -p backup
cp newtest.txt backup/
rm newtest.txt
[ -f backup/newtest.txt ] && echo "File exists in destination"
```

<div style="page-break-after: always;"></div>

## Practical 3: Shell Script using grep, awk and sed

```bash
#!/bin/bash
echo -e "apple 100\nbanana 50\napple 200\norange 80\ngrape 120" > data.txt
cat data.txt
grep "apple" data.txt
awk '$1=="apple" {sum+=$2} END {print sum}' data.txt
sed 's/apple/mango/g' data.txt
```

<div style="page-break-after: always;"></div>

## Practical 4: CPU Scheduling Algorithms (FCFS, SJF, Priority)

```c
#include <stdio.h>
#define N 5
int at[N]={0,1,2,3,4}, bt[N]={6,8,7,3,4}, pr[N]={3,1,4,2,5};

int key(int m,int i){return m==0?at[i]:m==1?bt[i]:pr[i];}

void run(int m){
    int done[N]={0}, ct[N], t=0, c=0;
    float st=0, sw=0;
    while(c<N){
        int s=-1;
        for(int i=0;i<N;i++)
            if(!done[i] && at[i]<=t && (s<0 || key(m,i)<key(m,s))) s=i;
        if(s<0){t++;continue;}
        t+=bt[s]; ct[s]=t; done[s]=1; c++;
    }
    printf("P AT BT PR CT TAT WT\n");
    for(int i=0;i<N;i++){
        int tat=ct[i]-at[i], wt=tat-bt[i];
        st+=tat; sw+=wt;
        printf("P%d %d  %d  %d  %d  %d  %d\n",i+1,at[i],bt[i],pr[i],ct[i],tat,wt);
    }
    printf("Avg TAT=%.2f Avg WT=%.2f\n\n",st/N,sw/N);
}

int main(){
    printf("FCFS\n"); run(0);
    printf("SJF\n"); run(1);
    printf("Priority\n"); run(2);
    return 0;
}
```

<div style="page-break-after: always;"></div>

## Practical 5: Page Replacement Algorithms (FIFO, Optimal, LRU)

```c
#include <stdio.h>
int r[]={7,0,1,2,0,3,0,4,2,3,0,3,2}, n=13, f=3;

int has(int *fr,int p){
    for(int i=0;i<f;i++) if(fr[i]==p) return 1;
    return 0;
}

int fifo(){
    int fr[10], k=0, pf=0;
    for(int i=0;i<f;i++) fr[i]=-1;
    for(int i=0;i<n;i++)
        if(!has(fr,r[i])){fr[k]=r[i]; k=(k+1)%f; pf++;}
    return pf;
}

int optimal(){
    int fr[10], pf=0;
    for(int i=0;i<f;i++) fr[i]=-1;
    for(int i=0;i<n;i++){
        if(has(fr,r[i])) continue;
        int v=-1, far=-1;
        for(int j=0;j<f;j++){
            if(fr[j]==-1){v=j;break;}
            int d=n;
            for(int k=i+1;k<n;k++) if(r[k]==fr[j]){d=k;break;}
            if(d>far){far=d;v=j;}
        }
        fr[v]=r[i]; pf++;
    }
    return pf;
}

int lru(){
    int fr[10], lu[10]={0}, pf=0;
    for(int i=0;i<f;i++) fr[i]=-1;
    for(int i=0;i<n;i++){
        int hit=-1;
        for(int j=0;j<f;j++) if(fr[j]==r[i]) hit=j;
        if(hit>=0){lu[hit]=i; continue;}
        int v=0;
        for(int j=0;j<f;j++){
            if(fr[j]==-1){v=j;break;}
            if(lu[j]<lu[v]) v=j;
        }
        fr[v]=r[i]; lu[v]=i; pf++;
    }
    return pf;
}

int main(){
    printf("FIFO page faults: %d\n",fifo());
    printf("Optimal page faults: %d\n",optimal());
    printf("LRU page faults: %d\n",lru());
    return 0;
}
```

<div style="page-break-after: always;"></div>

## Practical 6: Concurrent Programming (Threads, Processes, fork and vfork)

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
#include <pthread.h>

void *tf(void *a){
    printf("Thread running\n");
    return NULL;
}

int main(){
    pid_t p=fork();
    if(p==0){
        printf("fork child: pid=%d parent=%d\n",getpid(),getppid());
        _exit(0);
    }
    wait(NULL);
    printf("Parent resumed after fork child\n");

    pid_t v=vfork();
    if(v==0){
        printf("vfork child: pid=%d\n",getpid());
        _exit(0);
    }
    printf("Parent resumed after vfork child\n");

    pthread_t t;
    pthread_create(&t,NULL,tf,NULL);
    pthread_join(t,NULL);
    return 0;
}
```

<div style="page-break-after: always;"></div>

## Practical 7: Pthreads and Performance

```c
#include <stdio.h>
#include <pthread.h>
#include <time.h>
#define N 100000000L
#define T 4
long long part[T];

double now(){
    struct timespec t;
    clock_gettime(CLOCK_MONOTONIC,&t);
    return t.tv_sec+t.tv_nsec/1e9;
}

void *sum(void *a){
    long id=(long)a;
    long long s=0;
    for(long i=id*(N/T);i<(id+1)*(N/T);i++) s+=i;
    part[id]=s;
    return NULL;
}

int main(){
    double s=now();
    long long a=0;
    for(long i=0;i<N;i++) a+=i;
    double e1=now()-s;

    pthread_t th[T];
    s=now();
    for(long i=0;i<T;i++) pthread_create(&th[i],NULL,sum,(void*)i);
    long long b=0;
    for(int i=0;i<T;i++){pthread_join(th[i],NULL); b+=part[i];}
    double e2=now()-s;

    printf("Single thread: %lld in %.3f s\n",a,e1);
    printf("%d threads: %lld in %.3f s\n",T,b,e2);
    return 0;
}
```

<div style="page-break-after: always;"></div>

## Practical 8: Threads vs Processes (Matrix Multiplication)

```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
#include <sys/mman.h>
#include <sys/wait.h>
#include <time.h>
#define N 300
#define P 4
int A[N][N], B[N][N], CT[N][N];
int (*CP)[N];

double now(){
    struct timespec t;
    clock_gettime(CLOCK_MONOTONIC,&t);
    return t.tv_sec+t.tv_nsec/1e9;
}

void rows(int id,int (*C)[N]){
    for(int i=id*N/P;i<(id+1)*N/P;i++)
        for(int j=0;j<N;j++){
            int s=0;
            for(int k=0;k<N;k++) s+=A[i][k]*B[k][j];
            C[i][j]=s;
        }
}

void *work(void *a){
    rows((int)(long)a,CT);
    return NULL;
}

int main(){
    for(int i=0;i<N;i++)
        for(int j=0;j<N;j++){A[i][j]=(i+j)%10; B[i][j]=(i*j)%10;}

    pthread_t th[P];
    double s=now();
    for(long i=0;i<P;i++) pthread_create(&th[i],NULL,work,(void*)i);
    for(int i=0;i<P;i++) pthread_join(th[i],NULL);
    double et=now()-s;

    CP=mmap(NULL,sizeof(int)*N*N,PROT_READ|PROT_WRITE,MAP_SHARED|MAP_ANONYMOUS,-1,0);
    s=now();
    for(int i=0;i<P;i++)
        if(fork()==0){rows(i,CP); _exit(0);}
    for(int i=0;i<P;i++) wait(NULL);
    double ep=now()-s;

    printf("Threads:   %.3f s\n",et);
    printf("Processes: %.3f s\n",ep);
    printf("Results match: %s\n",CT[N-1][N-1]==CP[N-1][N-1]?"yes":"no");
    return 0;
}
```

<div style="page-break-after: always;"></div>

## Practical 9: Hyper Quick Sort, Merge Sort, Travelling Salesman Problem

```c
#include <stdio.h>
#include <pthread.h>
#define N 4

typedef struct{int *a,l,r,d;} Arg;

int part(int *a,int l,int r){
    int p=a[r], i=l-1, t;
    for(int j=l;j<r;j++)
        if(a[j]<p){i++; t=a[i]; a[i]=a[j]; a[j]=t;}
    t=a[i+1]; a[i+1]=a[r]; a[r]=t;
    return i+1;
}

void *pqs(void *x){
    Arg *g=x;
    if(g->l>=g->r) return NULL;
    int p=part(g->a,g->l,g->r);
    Arg L={g->a,g->l,p-1,g->d-1}, R={g->a,p+1,g->r,g->d-1};
    if(g->d>0){
        pthread_t t;
        pthread_create(&t,NULL,pqs,&L);
        pqs(&R);
        pthread_join(t,NULL);
    } else {
        pqs(&L);
        pqs(&R);
    }
    return NULL;
}

void ms(int *a,int l,int r){
    if(l>=r) return;
    int m=(l+r)/2, t[100], i=l, j=m+1, k=l;
    ms(a,l,m); ms(a,m+1,r);
    while(i<=m && j<=r) t[k++]=a[i]<=a[j]?a[i++]:a[j++];
    while(i<=m) t[k++]=a[i++];
    while(j<=r) t[k++]=a[j++];
    for(k=l;k<=r;k++) a[k]=t[k];
}

int d[N][N]={{0,10,15,20},{10,0,35,25},{15,35,0,30},{20,25,30,0}};
int best=1000000;

void tsp(int c,int vis,int cnt,int cost){
    if(cnt==N){
        if(cost+d[c][0]<best) best=cost+d[c][0];
        return;
    }
    for(int i=0;i<N;i++)
        if(!(vis&(1<<i))) tsp(i,vis|(1<<i),cnt+1,cost+d[c][i]);
}

void show(int *a,int n){
    for(int i=0;i<n;i++) printf("%d ",a[i]);
    printf("\n");
}

int main(){
    int a[]={38,27,43,3,9,82,10}, b[]={38,27,43,3,9,82,10}, n=7;
    Arg g={a,0,n-1,2};
    pqs(&g);
    printf("Parallel Quick Sort: "); show(a,n);
    ms(b,0,n-1);
    printf("Merge Sort: "); show(b,n);
    tsp(0,1,1,0);
    printf("TSP minimum cost: %d\n",best);
    return 0;
}
```

<div style="page-break-after: always;"></div>

## Practical 10: Synchronization Primitives (Semaphore, Locks, Condition Variables)

```c
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>
int cnt=0, ready=0;
pthread_mutex_t m=PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t c=PTHREAD_COND_INITIALIZER;
sem_t s;

void *inc(void *a){
    for(int i=0;i<100000;i++){
        pthread_mutex_lock(&m);
        cnt++;
        pthread_mutex_unlock(&m);
    }
    return NULL;
}

void *waiter(void *a){
    pthread_mutex_lock(&m);
    while(!ready) pthread_cond_wait(&c,&m);
    printf("Condition variable: condition met\n");
    pthread_mutex_unlock(&m);
    sem_post(&s);
    return NULL;
}

void *signaler(void *a){
    pthread_mutex_lock(&m);
    ready=1;
    pthread_cond_signal(&c);
    pthread_mutex_unlock(&m);
    return NULL;
}

int main(){
    pthread_t t1,t2,w,g;
    sem_init(&s,0,0);

    pthread_create(&t1,NULL,inc,NULL);
    pthread_create(&t2,NULL,inc,NULL);
    pthread_join(t1,NULL);
    pthread_join(t2,NULL);
    printf("Mutex: counter = %d\n",cnt);

    pthread_create(&w,NULL,waiter,NULL);
    pthread_create(&g,NULL,signaler,NULL);
    sem_wait(&s);
    printf("Semaphore: signal received\n");
    pthread_join(w,NULL);
    pthread_join(g,NULL);
    return 0;
}
```

<div style="page-break-after: always;"></div>

## Practical 11a: Producer-Consumer Problem

```c
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>
#define B 3
#define M 6
int buf[B], in=0, out=0;
sem_t empty, full;
pthread_mutex_t mx=PTHREAD_MUTEX_INITIALIZER;

void *prod(void *a){
    for(int i=1;i<=M;i++){
        sem_wait(&empty);
        pthread_mutex_lock(&mx);
        buf[in]=i; in=(in+1)%B;
        printf("Produced %d\n",i);
        pthread_mutex_unlock(&mx);
        sem_post(&full);
    }
    return NULL;
}

void *cons(void *a){
    for(int i=1;i<=M;i++){
        sem_wait(&full);
        pthread_mutex_lock(&mx);
        int x=buf[out]; out=(out+1)%B;
        printf("Consumed %d\n",x);
        pthread_mutex_unlock(&mx);
        sem_post(&empty);
    }
    return NULL;
}

int main(){
    pthread_t p,c;
    sem_init(&empty,0,B);
    sem_init(&full,0,0);
    pthread_create(&p,NULL,prod,NULL);
    pthread_create(&c,NULL,cons,NULL);
    pthread_join(p,NULL);
    pthread_join(c,NULL);
    return 0;
}
```

<div style="page-break-after: always;"></div>

## Practical 11b: Banker's Algorithm

```c
#include <stdio.h>
int main(){
    int n=5, m=3;
    int al[5][3]={{0,1,0},{2,0,0},{3,0,2},{2,1,1},{0,0,2}};
    int mx[5][3]={{7,5,3},{3,2,2},{9,0,2},{2,2,2},{4,3,3}};
    int av[3]={3,3,2}, nd[5][3], d[5]={0}, seq[5], c=0;

    for(int i=0;i<n;i++)
        for(int j=0;j<m;j++) nd[i][j]=mx[i][j]-al[i][j];

    while(c<n){
        int found=0;
        for(int i=0;i<n;i++){
            if(d[i]) continue;
            int ok=1;
            for(int j=0;j<m;j++) if(nd[i][j]>av[j]){ok=0;break;}
            if(ok){
                for(int j=0;j<m;j++) av[j]+=al[i][j];
                d[i]=1; seq[c++]=i; found=1;
            }
        }
        if(!found){printf("System is in unsafe state\n"); return 0;}
    }
    printf("Safe sequence: ");
    for(int i=0;i<n;i++) printf("P%d ",seq[i]);
    printf("\n");
    return 0;
}
```

<div style="page-break-after: always;"></div>

## Practical 12: Memory Allocation Algorithms (First Fit, Best Fit, Worst Fit)

```c
#include <stdio.h>
#define NB 5
#define NP 4
int blk[NB]={100,500,200,300,600}, prc[NP]={212,417,112,426};

void alloc(int m){
    int b[NB];
    for(int i=0;i<NB;i++) b[i]=blk[i];
    for(int i=0;i<NP;i++){
        int idx=-1;
        for(int j=0;j<NB;j++){
            if(b[j]>=prc[i]){
                if(idx<0 || (m==1&&b[j]<b[idx]) || (m==2&&b[j]>b[idx])) idx=j;
                if(m==0) break;
            }
        }
        if(idx>=0){b[idx]-=prc[i]; printf("Process %d (%d) -> Block %d\n",i+1,prc[i],idx+1);}
        else printf("Process %d (%d) -> Not allocated\n",i+1,prc[i]);
    }
    printf("\n");
}

int main(){
    printf("First Fit\n"); alloc(0);
    printf("Best Fit\n"); alloc(1);
    printf("Worst Fit\n"); alloc(2);
    return 0;
}
```

<div style="page-break-after: always;"></div>

## Practical 13: Disk Scheduling Algorithms (FCFS, SCAN, SSTF, C-SCAN)

```c
#include <stdio.h>
#include <stdlib.h>
#define N 8
#define D 200
int rq[N]={98,183,37,122,14,124,65,67}, h=53;

int main(){
    int t=0, p=h;
    for(int i=0;i<N;i++){t+=abs(rq[i]-p); p=rq[i];}
    printf("FCFS: %d\n",t);

    int v[N]={0}; t=0; p=h;
    for(int k=0;k<N;k++){
        int s=-1;
        for(int i=0;i<N;i++)
            if(!v[i] && (s<0 || abs(rq[i]-p)<abs(rq[s]-p))) s=i;
        v[s]=1; t+=abs(rq[s]-p); p=rq[s];
    }
    printf("SSTF: %d\n",t);

    int lo=D, hi=-1;
    for(int i=0;i<N;i++){
        if(rq[i]<h && rq[i]<lo) lo=rq[i];
        if(rq[i]<h && rq[i]>hi) hi=rq[i];
    }
    t=(D-1-h);
    if(lo<D) t+=(D-1)-lo;
    printf("SCAN: %d\n",t);

    t=(D-1-h);
    if(hi>=0) t+=(D-1)+hi;
    printf("C-SCAN: %d\n",t);
    return 0;
}
```

<div style="page-break-after: always;"></div>

## Practical 14: Kernel Reconfiguration, Device Drivers, System Administration, Utilities and OS Performance Tuning

Writing a utility (Bash script):

```bash
#!/bin/bash

echo "================================"
echo "       SYSTEM INFORMATION"
echo "================================"

echo "1. Operating System:"
uname -o

echo ""
echo "2. Kernel Version:"
uname -r

echo ""
echo "3. CPU Information:"
lscpu | grep "Model name"

echo ""
echo "4. Memory Usage:"
free -h

echo ""
echo "5. Disk Usage:"
df -h

echo ""
echo "6. System Uptime:"
uptime

echo ""
echo "7. Logged-in User:"
whoami

echo ""
echo "8. Running Processes:"
ps aux | head -5

echo ""
echo "9. Performance (vmstat):"
vmstat 1 3

echo ""
echo "10. Swappiness:"
sysctl vm.swappiness

echo "================================"
echo "       END OF REPORT"
echo "================================"
```

System administration and OS performance tuning (Linux terminal):

```bash
uname -a
lsmod | head
free -m
df -h
ps aux | head
top -bn1 | head -n 15
vmstat 1 3
sudo useradd testuser
sudo userdel testuser
sysctl vm.swappiness
sudo sysctl -w vm.swappiness=10
sleep 60 &
renice -n 10 -p $!
ps -o pid,ni,comm -p $!
kill $!
```

Kernel reconfiguration (Ubuntu/Debian):

```bash
sudo apt update
sudo apt install build-essential libncurses-dev flex bison libssl-dev libelf-dev bc wget
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.6.tar.xz
tar -xf linux-6.6.tar.xz
cd linux-6.6
cp /boot/config-$(uname -r) .config
scripts/config --disable SYSTEM_TRUSTED_KEYS --disable SYSTEM_REVOCATION_KEYS
make menuconfig
make -j$(nproc)
sudo make modules_install
sudo make install
sudo reboot
```

Device driver: character device (`hello.c`):

```c
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/string.h>
#include <linux/uaccess.h>

static int major;
static const char msg[]="Hello from driver\n";

static ssize_t dev_read(struct file *f,char __user *buf,size_t len,loff_t *off){
    size_t n=strlen(msg);
    if(*off>=n) return 0;
    if(len>n-*off) len=n-*off;
    if(copy_to_user(buf,msg+*off,len)) return -EFAULT;
    *off+=len;
    return len;
}

static struct file_operations fops={
    .owner=THIS_MODULE,
    .read=dev_read,
};

static int __init hello_init(void){
    major=register_chrdev(0,"hello",&fops);
    if(major<0) return major;
    printk(KERN_INFO "hello: major number %d\n",major);
    return 0;
}

static void __exit hello_exit(void){
    unregister_chrdev(major,"hello");
    printk(KERN_INFO "hello: unloaded\n");
}

module_init(hello_init);
module_exit(hello_exit);
MODULE_LICENSE("GPL");
```

`Makefile` (indent the make lines with a TAB):

```make
obj-m += hello.o
all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

Build, load and test (replace `MAJOR` with the number shown by `dmesg`):

```bash
sudo apt install build-essential linux-headers-$(uname -r)
make
sudo insmod hello.ko
sudo dmesg | tail -n 3
sudo mknod /dev/hello c MAJOR 0
sudo cat /dev/hello
sudo rmmod hello
sudo rm /dev/hello
```
