# Operating Systems Laboratory: Practical Codes

<div style="page-break-after: always;"></div>

## Practical No. 1: Hands-On UNIX Commands

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

## Practical No. 2: Shell Programming for File Handling

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

## Practical No. 3: Shell Script using grep, awk and sed

```bash
#!/bin/bash
echo -e "apple 100\nbanana 50\napple 200\norange 80\ngrape 120" > data.txt
cat data.txt
grep "apple" data.txt
awk '$1=="apple" {sum+=$2} END {print sum}' data.txt
sed 's/apple/mango/g' data.txt
```

<div style="page-break-after: always;"></div>

## Practical No. 4: CPU Scheduling Algorithms (FCFS, SJF, Priority)

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

## Practical No. 5: Page Replacement Algorithms (FIFO, Optimal, LRU)

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

Expected: FIFO 10, Optimal 7, LRU 9.

<div style="page-break-after: always;"></div>

## Practical No. 6: Quick Sort, Merge Sort and Travelling Salesman Problem

```c
#include <stdio.h>
#define N 4

void qs(int *a,int l,int r){
    if(l>=r) return;
    int p=a[r], i=l-1, t;
    for(int j=l;j<r;j++)
        if(a[j]<p){i++; t=a[i]; a[i]=a[j]; a[j]=t;}
    t=a[i+1]; a[i+1]=a[r]; a[r]=t;
    qs(a,l,i); qs(a,i+2,r);
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
    qs(a,0,n-1);
    printf("Quick Sort: "); show(a,n);
    ms(b,0,n-1);
    printf("Merge Sort: "); show(b,n);
    tsp(0,1,1,0);
    printf("TSP minimum cost: %d\n",best);
    return 0;
}
```

Expected: TSP minimum cost 80.

<div style="page-break-after: always;"></div>

## Practical No. 7a: Producer-Consumer Problem

Compile with `-pthread`.

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

## Practical No. 7b: Banker's Algorithm

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

Expected: Safe sequence P1 P3 P4 P0 P2.

<div style="page-break-after: always;"></div>

## Practical No. 8: Memory Allocation Algorithms (First Fit, Best Fit, Worst Fit)

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

## Practical No. 9: Disk Scheduling Algorithms (FCFS, SSTF, SCAN, C-SCAN)

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

Expected: FCFS 640, SSTF 236, SCAN 331, C-SCAN 382.<img width="1917" height="770" alt="image" src="https://github.com/user-attachments/assets/88b41ac2-d087-44e9-bc36-91088b2437bc" />
