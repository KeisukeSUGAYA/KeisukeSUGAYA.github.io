---
layout: post
title: MPIによる並列プログラミング（基本的な通信）
tags: MPI
---

各プロセスが所有する整数型のデータの総和を計算します．プログラムは Fortran で書かれています．

## 目次
- 計算内容
- サブルーチンの構成
- 双方向通信
- 集団通信
- 片側通信
- 参考文献
- プログラム全体

## 計算内容
本稿では，RANK が i のプロセスをプロセスi と呼ぶことにします．プロセスi は，整数型のデータ x_i を持っており，x_i=i です．また総プロセス数はNです．本稿では，x_i の総和を計算します．  
具体的に，N=4の場合を考えます．プロセス0は x_0 = 0，プロセス1は x_1 = 1，プロセス2は x_2 = 2，プロセス3は x_3 = 3 を持っています．このx_0からx_3の総和を計算することを目的とします．  
この計算は `MPI_Reduce` を利用すると簡単に求まるのですが，本稿ではいろいろなMPIの通信を利用し，計算してみます．

## サブルーチンの構成
プログラムの全体は，本稿の一番下に載せてあります．各サブルーチンの構成は

1. 時間計測（開始）
2. x_iの総和を計算
3. 時間計測（終了）
4. RANK=0のプロセスが，計算時間と計算結果を表示  

となっています．以下では，x_iの総和計算に着目します．

## 双方向通信
### send/recv
ブロッキング通信である， `MPI_SEND` と `MPI_RECV` を利用します．  
RANK /= 0 の場合は，RANK = 0 に向けてx_i を送信します．またRANK == 0 の場合は，RANK /= 0 のプロセスから送られてくるx_iを受信し，変数 `total` に足し合わせています．
```
   if(MYRANK == 0)then
      total = MYRANK
      do i = 1, SIZE-1  ! exclude rank0
        !*** recieve data, size of recive data, data type, rank of send process, tag, comm_world, status, ierr
        call MPI_RECV(recv,1,MPI_INTEGER,i,i,MPI_COMM_WORLD,status,ierr)
        total = total + recv
      enddo
    else
      send = MYRANK
      !*** send data, size of send data, data type, rank of recieve process, tag, ierr
      call MPI_SEND(send,1,MPI_INTEGER,0,MYRANK,MPI_COMM_WORLD,ierr)
    endif
```

### isend/irecv
ブロッキング通信である， `MPI_ISEND` と `MPI_IRECV` を利用します．  やっていることは，ブロッキング通信の send/recv とほとんど同じです．

```
    allocate(status(MPI_STATUS_SIZE,SIZE-1),request(SIZE-1))
    if(MYRANK == 0)then
      allocate(recv(SIZE-1))
      do i = 1, SIZE-1  ! exclude rank0
        call MPI_IRECV(recv(i),1,MPI_INTEGER,i,i,MPI_COMM_WORLD,request(i),ierr)
      enddo
      call MPI_WAITALL(SIZE-1,request(:), status(:,:), ierr)
      total = MYRANK + sum(recv(1:SIZE-1))
    else
      send = MYRANK
      call MPI_ISEND(send,1,MPI_INTEGER,0,MYRANK,MPI_COMM_WORLD,request(MYRANK),ierr)
      call MPI_WAITALL(1,request(MYRANK), status(:,:), ierr)
    endif
    deallocate(status,request)
```

### sendrecv
send と recv がひとつになった，`MPI_SENDRECV` を利用します．また，以下では総プロセス数がNの場合を考えます．  
はじめに，プロセス0~プロセスNが `MPI_SENDRECV` を呼びます．．プロセスiはプロセスi-1にx_iを送信し，またプロセスi+1からx_i+1を受信します．またプロセス0はどのプロセスにもx_0を送信せず，プロセスNはどのプロセスからも受信しません（送受信先を `MPI_PROC_NULL` とします）．  
次に，プロセス0~プロセスN-1が `MPI_SENDRECV` を呼びます．通信の内容は，先ほどと同様に，i-1に送信し，i+1から受信します．  
これをN回繰り返すと，x_Nの情報が，プロセスNからプロセスN-1，プロセスN-2...と伝わっていき，N-1回の通信でプロセス0に届きます．  
またプロセス0は，x_1，x_2...x_N の情報を，N-1回の通信で受信できます．ここで，送られてきたx_iを利用し， `total` に足し合わせておくことで，x_i の総和を計算します．

```
    total = 0
    src   = MYRANK+1
    dst   = MYRANK-1
    if(MYRANK==0)dst = MPI_PROC_NULL
    recv = MYRANK
    do i = 1, SIZE-1
      if(MYRANK <=(SIZE-1)-(i-1))then
        if(MYRANK==(SIZE-1)-(i-1))src = MPI_PROC_NULL
        send = recv
        call MPI_SENDRECV(send,1,MPI_INTEGER,dst,MYRANK,    &
                       &  recv,1,MPI_INTEGER,src,MYRANK+1,  &
                       &  MPI_COMM_WORLD,status,ierr)
        total = total + recv
      endif
    enddo
```

## 集団通信
### reduce
集団通信の `MPI_REDUCE` を利用すると，簡単に総和を計算できます．ここでは，受信先をプロセス0とします．
```
    send = MYRANK
    !*** send data, recieve data, # data, data type, operator, recieve process ,comm world, error
    call MPI_REDUCE(send,total,1,MPI_INTEGER,MPI_SUM,0,MPI_COMM_WORLD,ierr)
```

### allreduce
上の reduce と同様に，`MPI_ALLREDUCE` でも簡単に総和計算できます．

```
    send = MYRANK
    !*** send data, recieve data, # data, data type, operator, recieve process ,comm world, error
    call MPI_ALLREDUCE(send,total,1,MPI_INTEGER,MPI_SUM,MPI_COMM_WORLD,ierr)
```


### gather
`MPI_GATHER` を利用すると，x_i を大きさがNの配列 __x__ にまとめることができます．あとは， __x__ の総和を計算すれば良いです．

```
    allocate(send(1),recv(SIZE))
    send(1) = MYRANK
    !*** send data, # data (each process), data type, recieve data, # data, data type, resieve process, comm_world, error
    call MPI_GATHER(send(1),1,MPI_INTEGER,recv(1),1,MPI_INTEGER,0,MPI_COMM_WORLD,ierr)
    if(MYRANK==0) total = sum(recv)
    deallocate(send,recv)
```


### allgather
gather と同じです．
```
    allocate(send(1),recv(SIZE))
    send(1) = MYRANK
    !*** send data, # data (each process), data type, recieve data, # data, data type, comm_world, error
    !*** no recieve process
    call MPI_ALLGATHER(send(1),1,MPI_INTEGER,recv(1),1,MPI_INTEGER,MPI_COMM_WORLD,ierr)
    if(MYRANK==0) total = sum(recv)
    deallocate(send,recv)
```


### alltoall
`MPI_ALLTOALL` を利用すると，行列の転置が可能です．
```
    allocate(send(SIZE),recv(SIZE))
    send(1) = MYRANK
    !*** send data, # data (each process), data type, recieve data, # data, data type, comm_world, error
    !*** no recieve process
    call MPI_ALLTOALL(send(1),1,MPI_INTEGER,recv(1),1,MPI_INTEGER,MPI_COMM_WORLD,ierr)

    if(MYRANK==0) total = sum(recv)
    deallocate(send,recv)
```

## bcast
`MPI_BCAST` は，あるプロセスの情報を他のプロセスに送信します．総プロセス数がNであれば，`MPI_BCAST` を N 回呼ぶことで，すべてのプロセスにx_iの情報が伝わります．

```
    total = 0
    do i = 0, SIZE-1
      sendrecv = MYRANK
      call MPI_Bcast(sendrecv,1,MPI_INTEGER,i,MPI_COMM_WORLD,ierr)
      total = total + sendrecv
    enddo
```

### scatter
`MPI_SCATTER` を利用すると，行列の転置が可能です．alltoallと似ています．

```
    allocate(send(SIZE),recv(SIZE))
    total = 0
    do i = 0, SIZE-1
      send(1) = MYRANK
      call MPI_SCATTER(send(1),1,MPI_INTEGER,recv(1),1,MPI_INTEGER,i,MPI_COMM_WORLD,ierr)
      total = total+recv(1)
    enddo

    deallocate(send,recv)
```

### scan
`MPI_SCAN` は，RANKが自分以上のプロセスに情報を送信します（自分含む）．またRANKが自分以下のプロセスからの情報を受信し，受信した情報に対し演算をします（自分含む）．このため，プロセス0は自分が送信した情報，つまりx_0を受信します．また演算に `MPI_SUM` を指定することで，プロセスNはx_0からx_Nの総和を計算します．下のプログラムでは，プロセスNからプロセス0に，総和計算の結果を送っています．

```
    send = MYRANK
    call MPI_SCAN(send,recv,1,MPI_INTEGER,MPI_SUM,MPI_COMM_WORLD,ierr)

    if(MYRANK==0)then
      call MPI_RECV(recv,1,MPI_INTEGER,SIZE-1,SIZE-1,MPI_COMM_WORLD,status,ierr)
      total = recv
    elseif(MYRANK==SIZE-1)then
      send = recv
      call MPI_SEND(send,1,MPI_INTEGER,0,SIZE-1,MPI_COMM_WORLD,ierr)
    endif
```

## 片側通信
レファレンスなどを読んでプログラムを書いたのですが，間違いがある気がします．以下の内容は，あまり信用しないほうがいいでしょう．

### get
プロセス0が，0を除くプロセスiの公開されている領域にアクセスし，情報を取得します．0を除くプロセスiは，あらかじめ公開領域にx_iを置いておきます．

```
    !*** create window, initialize
    call MPI_TYPE_GET_EXTENT(MPI_INTEGER, lb, sizeofinteger, ierr)
    disp_int = sizeofinteger 
    nbytes   = 1*sizeofinteger
    allocate(base(1),local(1))
    total = 0
    base(1) = 0
    call MPI_WIN_CREATE(base,nbytes,disp_int,MPI_INFO_NULL,MPI_COMM_WORLD,win,ierr)
    base(1) = MYRANK
    
    disp_aint = 0
    if(MYRANK == 0)then
      total = MYRANK
      do i = 1, SIZE-1
        !*** Passive Remote Memory Access (RMA) synchronization. Begin an RMA epoch at the target process.
        !***  lock_type (MPI_LOCK_EXCLUSIVE / MPI_LOCK_SHARED), rank of locked window, assert(zero may be used as a default), window, error
        call MPI_WIN_LOCK(MPI_LOCK_EXCLUSIVE, i, 0, win, ierr)

        !*** recieve data, # recieve data, data type, Process accessed by MPI_get, data address, # data, data type, window, error
        call MPI_GET(local(1),1,MPI_INTEGER,i,disp_aint,1,MPI_INTEGER,win,ierr)

        !*** Completes an RMA access at the target process
        !*** rank of window, window, error
        call MPI_WIN_UNLOCK(i, win, ierr)

        total = total + local(1)
      enddo
    endif

    !*** Finalize window
    call MPI_WIN_FREE(win,ierr)
    deallocate(base,local)
```

### put
0を除くプロセスiは，プロセス0の公開されている領域にアクセスし，x_iを置きます．またプロセス0は，プロセス1~プロセスNが情報を置けるように，公開領域を設定します．

```
    !*** create window, initialize
    call MPI_TYPE_GET_EXTENT(MPI_INTEGER, lb, sizeofinteger, ierr)
    nbytes   = SIZE*sizeofinteger
    disp_int = sizeofinteger 
    allocate(base(SIZE),local(1))
    base(1:SIZE) = 0
    total = 0
    call MPI_WIN_CREATE(base,nbytes,disp_int,MPI_INFO_NULL,MPI_COMM_WORLD,win,ierr)

    if(MYRANK==0)then
      base(1) = MYRANK
    else
      local(1) = MYRANK
      disp_aint = MYRANK
      !*** Passive Remote Memory Access (RMA) synchronization. Begin an RMA epoch at the target process.
      !***  lock_type (MPI_LOCK_EXCLUSIVE / MPI_LOCK_SHARED), rank of locked window, assert(zero may be used as a default), window, error
      call MPI_WIN_LOCK(MPI_LOCK_EXCLUSIVE, 0, 0, win, ierr)

      !*** put data, # put data, data type, Process accessed by MPI_put, data address, window, error
      call MPI_PUT(local(1),1,MPI_INTEGER,0,disp_aint,1,MPI_INTEGER,win,ierr)

      !*** Completes an RMA access at the target process
      !*** rank of window, window, error
      call MPI_WIN_UNLOCK(0, win, ierr)
    endif

    !*** Finalize window
    call MPI_WIN_FREE(win,ierr)
    if(MYRANK==0) total = sum(base(1:SIZE))
    deallocate(base,local)
```

## 参考文献
本稿を作成するために，以下のサイトを参考にしました．
Ref. 1 : https://www.mpi-forum.org/
Ref. 2 : https://www.mpich.org/
Ref. 3 : http://www.hpci-office.jp/pages/seminar_texts
Ref. 4 : http://www.cv.titech.ac.jp/~hiro-lab/study/mpi_reference/index.html


## プログラム全体
コンパイルは， `mpif90 mpi_sum.f90 -o mpi_sum` です．


```
!***************************************************************
!*** Ref. 1 : https://www.mpi-forum.org/
!*** Ref. 2 : https://www.mpich.org/
!*** Ref. 3 : http://www.hpci-office.jp/pages/seminar_texts
!*** Ref. 4 : http://www.cv.titech.ac.jp/~hiro-lab/study/mpi_reference/index.html
!***************************************************************

PROGRAM main
  implicit none
  include 'mpif.h'
  integer :: SIZE, MYRANK, IERR

  call MPI_INIT(IERR)
  call MPI_COMM_SIZE(MPI_COMM_WORLD, SIZE, IERR)
  call MPI_COMM_RANK(MPI_COMM_WORLD, MYRANK, IERR)
  if(MYRANK == 0)then
    write(*,"(A)")              "!*********************************************************!"
    write(*,"(A,i4)")           "    MPI_COMM_SIZE    : ", SIZE
    write(*,"(A,i4,A,i4,A,i4)") "    SUM ", 0, " to ", SIZE-1, " : ", (SIZE-1)*SIZE/2
    write(*,"(A)")              "!*********************************************************!"
  endif
  call MPI_BARRIER(MPI_COMM_WORLD, IERR)

  !*** Point-to-Point Communication
  call PointToPoint_send_recv
  call PointToPoint_isend_irecv
  call PointToPoint_sendrecv ! combine

  !*** Collective Communication
  call Collective_reduce
  call Collective_allreduce
  call Collective_gather
  call Collective_allgather
  call Collective_alltoall
  call Collective_bcast
  call Collective_scatter
  call Collective_scan

  !*** One-Sided Communications
  call OneSided_get
  call OneSided_put

  call MPI_FINALIZE (IERR)

contains

!***************************************************************
!***************************************************************
  subroutine PointToPoint_send_recv
    double precision :: stime, etime
    integer          :: total, send, recv
    integer          :: i
    integer          :: status(MPI_STATUS_SIZE), ierr

    !*** Start testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) stime=MPI_Wtime()
    !***

    if(MYRANK == 0)then
      total = MYRANK
      do i = 1, SIZE-1  ! exclude rank0
        !*** recieve data, size of recive data, data type, rank of send process, tag, comm_world, status, ierr
        call MPI_RECV(recv,1,MPI_INTEGER,i,i,MPI_COMM_WORLD,status,ierr)
        total = total + recv
      enddo
    else
      send = MYRANK
      !*** send data, size of send data, data type, rank of recieve process, tag, ierr
      call MPI_SEND(send,1,MPI_INTEGER,0,MYRANK,MPI_COMM_WORLD,ierr)
    endif

    !*** End testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) etime=MPI_Wtime()
    if(MYRANK==0)then
      write(*,"(A)") ""
      write(*,"(A)") "send/recv"
      write(*,"(A,i4)")   "result : ", total
      write(*,"(A,E15.7)") "time   : ", etime-stime
    endif
    !***
  end subroutine PointToPoint_send_recv


!***************************************************************
!***************************************************************
  subroutine PointToPoint_isend_irecv
    double precision    :: stime, etime
    integer             :: total
    integer             :: send
    integer,allocatable :: recv(:)
    integer             :: i
    integer,allocatable :: status(:,:), request(:)
    integer             :: ierr

    !*** Start testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) stime=MPI_Wtime()
    !***

    allocate(status(MPI_STATUS_SIZE,SIZE-1),request(SIZE-1))
    if(MYRANK == 0)then
      allocate(recv(SIZE-1))
      do i = 1, SIZE-1  ! exclude rank0
        call MPI_IRECV(recv(i),1,MPI_INTEGER,i,i,MPI_COMM_WORLD,request(i),ierr)
      enddo
      call MPI_WAITALL(SIZE-1,request(:), status(:,:), ierr)
      total = MYRANK + sum(recv(1:SIZE-1))
    else
      send = MYRANK
      call MPI_ISEND(send,1,MPI_INTEGER,0,MYRANK,MPI_COMM_WORLD,request(MYRANK),ierr)
      call MPI_WAITALL(1,request(MYRANK), status(:,:), ierr)
    endif
    deallocate(status,request)

    !*** End testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) etime=MPI_Wtime()
    if(MYRANK==0)then
      write(*,"(A)") ""
      write(*,"(A)") "isend/irecv"
      write(*,"(A,i4)")   "result : ", total
      write(*,"(A,E15.7)") "time   : ", etime-stime
    endif
    !***
  end subroutine PointToPoint_isend_irecv


!***************************************************************
!***************************************************************
  subroutine PointToPoint_sendrecv
    double precision :: stime, etime
    integer          :: total
    integer          :: send, recv
    integer          :: dst, src
    integer          :: i
    integer          :: status(MPI_STATUS_SIZE)
    integer          :: ierr

    !*** Start testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) stime=MPI_Wtime()
    !***

    total = 0
    src   = MYRANK+1
    dst   = MYRANK-1
    if(MYRANK==0)dst = MPI_PROC_NULL
    recv = MYRANK
    do i = 1, SIZE-1
      if(MYRANK <=(SIZE-1)-(i-1))then
        if(MYRANK==(SIZE-1)-(i-1))src = MPI_PROC_NULL
        send = recv
        call MPI_SENDRECV(send,1,MPI_INTEGER,dst,MYRANK,    &
                       &  recv,1,MPI_INTEGER,src,MYRANK+1,  &
                       &  MPI_COMM_WORLD,status,ierr)
        total = total + recv
      endif
    enddo

    !*** End testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) etime=MPI_Wtime()
    if(MYRANK==0)then
      write(*,"(A)") ""
      write(*,"(A)") "sendrecv"
      write(*,"(A,i4)")   "result : ", total
      write(*,"(A,E15.7)") "time   : ", etime-stime
    endif
    !***
  end subroutine PointToPoint_sendrecv


!***************************************************************
!***************************************************************
  subroutine Collective_reduce
    double precision    :: stime, etime
    integer             :: send, total
    integer             :: ierr

    !*** Start testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) stime=MPI_Wtime()
    !***

    send = MYRANK
    !*** send data, recieve data, # data, data type, operator, recieve process ,comm world, error
    call MPI_REDUCE(send,total,1,MPI_INTEGER,MPI_SUM,0,MPI_COMM_WORLD,ierr)

    !*** End testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) etime=MPI_Wtime()
    if(MYRANK==0)then
      write(*,"(A)") ""
      write(*,"(A)") "reduce"
      write(*,"(A,i4)")   "result : ", total
      write(*,"(A,E15.7)") "time   : ", etime-stime
    endif
    !***
  end subroutine Collective_reduce


!***************************************************************
!***************************************************************
  subroutine Collective_allreduce
    double precision    :: stime, etime
    integer             :: send, total
    integer             :: ierr

    !*** Start testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) stime=MPI_Wtime()
    !***

    send = MYRANK
    !*** send data, recieve data, # data, data type, operator, recieve process ,comm world, error
    call MPI_ALLREDUCE(send,total,1,MPI_INTEGER,MPI_SUM,MPI_COMM_WORLD,ierr)

    !*** End testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) etime=MPI_Wtime()
    if(MYRANK==0)then
      write(*,"(A)") ""
      write(*,"(A)") "allreduce"
      write(*,"(A,i4)")   "result : ", total
      write(*,"(A,E15.7)") "time   : ", etime-stime
    endif
    !***
  end subroutine Collective_allreduce


!***************************************************************
!***************************************************************
  subroutine Collective_gather
    double precision    :: stime, etime
    integer             :: total
    integer,allocatable :: send(:), recv(:)
    integer             :: ierr

    !*** Start testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) stime=MPI_Wtime()
    !***

    allocate(send(1),recv(SIZE))
    send(1) = MYRANK
    !*** send data, # data (each process), data type, recieve data, # data, data type, resieve process, comm_world, error
    call MPI_GATHER(send(1),1,MPI_INTEGER,recv(1),1,MPI_INTEGER,0,MPI_COMM_WORLD,ierr)
    if(MYRANK==0) total = sum(recv)
    deallocate(send,recv)

    !*** End testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) etime=MPI_Wtime()
    if(MYRANK==0)then
      write(*,"(A)") ""
      write(*,"(A)") "gather"
      write(*,"(A,i4)")   "result : ", total
      write(*,"(A,E15.7)") "time   : ", etime-stime
    endif
    !***
  end subroutine Collective_gather


!***************************************************************
!***************************************************************
  subroutine Collective_allgather
    double precision    :: stime, etime
    integer             :: total
    integer,allocatable :: send(:), recv(:)
    integer             :: ierr

    !*** Start testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) stime=MPI_Wtime()
    !***

    allocate(send(1),recv(SIZE))
    send(1) = MYRANK
    !*** send data, # data (each process), data type, recieve data, # data, data type, comm_world, error
    !*** no recieve process
    call MPI_ALLGATHER(send(1),1,MPI_INTEGER,recv(1),1,MPI_INTEGER,MPI_COMM_WORLD,ierr)
    if(MYRANK==0) total = sum(recv)
    deallocate(send,recv)

    !*** End testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) etime=MPI_Wtime()
    if(MYRANK==0)then
      write(*,"(A)") ""
      write(*,"(A)") "allgather"
      write(*,"(A,i4)")   "result : ", total
      write(*,"(A,E15.7)") "time   : ", etime-stime
    endif
    !***
  end subroutine Collective_allgather


!***************************************************************
!***************************************************************
  subroutine Collective_alltoall
    double precision    :: stime, etime
    integer             :: total
    integer,allocatable :: send(:), recv(:)
    integer             :: ierr

    !*** Start testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) stime=MPI_Wtime()
    !***

    allocate(send(SIZE),recv(SIZE))
    send(1) = MYRANK
    !*** send data, # data (each process), data type, recieve data, # data, data type, comm_world, error
    !*** no recieve process
    call MPI_ALLTOALL(send(1),1,MPI_INTEGER,recv(1),1,MPI_INTEGER,MPI_COMM_WORLD,ierr)

    if(MYRANK==0) total = sum(recv)
    deallocate(send,recv)

    !*** End testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) etime=MPI_Wtime()
    if(MYRANK==0)then
      write(*,"(A)") ""
      write(*,"(A)") "alltoall"
      write(*,"(A,i4)")   "result : ", total
      write(*,"(A,E15.7)") "time   : ", etime-stime
    endif
    !***
  end subroutine Collective_alltoall


!***************************************************************
!***************************************************************
  subroutine Collective_bcast
    double precision :: stime, etime
    integer          :: total, sendrecv
    integer          :: i
    integer          :: ierr

    !*** Start testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) stime=MPI_Wtime()
    !***

    total = 0
    do i = 0, SIZE-1
      sendrecv = MYRANK
      call MPI_Bcast(sendrecv,1,MPI_INTEGER,i,MPI_COMM_WORLD,ierr)
      total = total + sendrecv
    enddo

    !*** End testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) etime=MPI_Wtime()
    if(MYRANK==0)then
      write(*,"(A)") ""
      write(*,"(A)") "bcast"
      write(*,"(A,i4)")   "result : ", total
      write(*,"(A,E15.7)") "time   : ", etime-stime
    endif
    !***
  end subroutine Collective_bcast


!***************************************************************
!***************************************************************
  subroutine Collective_scatter
    double precision    :: stime, etime
    integer             :: total, sendrecv
    integer,allocatable :: send(:),recv(:)
    integer             :: i
    integer             :: ierr

    !*** Start testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) stime=MPI_Wtime()
    !***

    allocate(send(SIZE),recv(SIZE))
    total = 0
    do i = 0, SIZE-1
      send(1) = MYRANK
      call MPI_SCATTER(send(1),1,MPI_INTEGER,recv(1),1,MPI_INTEGER,i,MPI_COMM_WORLD,ierr)
      total = total+recv(1)
    enddo

    deallocate(send,recv)


    !*** End testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) etime=MPI_Wtime()
    if(MYRANK==0)then
      write(*,"(A)") ""
      write(*,"(A)") "scatter"
      write(*,"(A,i4)")   "result : ", total
      write(*,"(A,E15.7)") "time   : ", etime-stime
    endif
    !***
  end subroutine Collective_scatter


!***************************************************************
!***************************************************************
  subroutine Collective_scan
    double precision    :: stime, etime
    integer             :: total, send, recv
    integer             :: ierr, status(MPI_STATUS_SIZE)

    !*** Start testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) stime=MPI_Wtime()
    !***

    send = MYRANK
    call MPI_SCAN(send,recv,1,MPI_INTEGER,MPI_SUM,MPI_COMM_WORLD,ierr)

    if(MYRANK==0)then
      call MPI_RECV(recv,1,MPI_INTEGER,SIZE-1,SIZE-1,MPI_COMM_WORLD,status,ierr)
      total = recv
    elseif(MYRANK==SIZE-1)then
      send = recv
      call MPI_SEND(send,1,MPI_INTEGER,0,SIZE-1,MPI_COMM_WORLD,ierr)
    endif

    !*** End testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) etime=MPI_Wtime()
    if(MYRANK==0)then
      write(*,"(A)") ""
      write(*,"(A)") "scan"
      write(*,"(A,i4)")   "result : ", total
      write(*,"(A,E15.7)") "time   : ", etime-stime
    endif
    !***
  end subroutine Collective_scan


!***************************************************************
!***************************************************************
  subroutine OneSided_get
    double precision               :: stime, etime
    integer,pointer                :: base(:), local(:)
    integer                        :: total
    integer                        :: i
    integer                        :: disp_int, win
    integer(kind=MPI_ADDRESS_KIND) :: nbytes, disp_aint, lb, sizeofinteger
    integer                        :: ierr, status(MPI_STATUS_SIZE)

    !*** Start testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) stime=MPI_Wtime()
    !***

    !*** create window, initialize
    call MPI_TYPE_GET_EXTENT(MPI_INTEGER, lb, sizeofinteger, ierr)
    disp_int = sizeofinteger 
    nbytes   = 1*sizeofinteger
    allocate(base(1),local(1))
    total = 0
    base(1) = 0
    call MPI_WIN_CREATE(base,nbytes,disp_int,MPI_INFO_NULL,MPI_COMM_WORLD,win,ierr)
    base(1) = MYRANK
    
    disp_aint = 0
    if(MYRANK == 0)then
      total = MYRANK
      do i = 1, SIZE-1
        !*** Passive Remote Memory Access (RMA) synchronization. Begin an RMA epoch at the target process.
        !***  lock_type (MPI_LOCK_EXCLUSIVE / MPI_LOCK_SHARED), rank of locked window, assert(zero may be used as a default), window, error
        call MPI_WIN_LOCK(MPI_LOCK_EXCLUSIVE, i, 0, win, ierr)

        !*** recieve data, # recieve data, data type, Process accessed by MPI_get, data address, # data, data type, window, error
        call MPI_GET(local(1),1,MPI_INTEGER,i,disp_aint,1,MPI_INTEGER,win,ierr)

        !*** Completes an RMA access at the target process
        !*** rank of window, window, error
        call MPI_WIN_UNLOCK(i, win, ierr)

        total = total + local(1)
      enddo
    endif

    !*** Finalize window
    call MPI_WIN_FREE(win,ierr)
    deallocate(base,local)

    !*** End testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) etime=MPI_Wtime()
    if(MYRANK==0)then
      write(*,"(A)") ""
      write(*,"(A)") "get"
      write(*,"(A,i4)")   "result : ", total
      write(*,"(A,E15.7)") "time   : ", etime-stime
    endif
    !***
  end subroutine OneSided_get


!***************************************************************
!***************************************************************
  subroutine OneSided_put
    double precision               :: stime, etime
    integer,pointer                :: base(:), local(:)
    integer                        :: total
    integer                        :: i
    integer                        :: disp_int, win
    integer(kind=MPI_ADDRESS_KIND) :: nbytes, disp_aint, lb, sizeofinteger
    integer                        :: ierr, status(MPI_STATUS_SIZE)

    !*** Start testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) stime=MPI_Wtime()
    !***

    !*** create window, initialize
    call MPI_TYPE_GET_EXTENT(MPI_INTEGER, lb, sizeofinteger, ierr)
    nbytes   = SIZE*sizeofinteger
    disp_int = sizeofinteger 
    allocate(base(SIZE),local(1))
    base(1:SIZE) = 0
    total = 0
    call MPI_WIN_CREATE(base,nbytes,disp_int,MPI_INFO_NULL,MPI_COMM_WORLD,win,ierr)

    if(MYRANK==0)then
      base(1) = MYRANK
    else
      local(1) = MYRANK
      disp_aint = MYRANK
      !*** Passive Remote Memory Access (RMA) synchronization. Begin an RMA epoch at the target process.
      !***  lock_type (MPI_LOCK_EXCLUSIVE / MPI_LOCK_SHARED), rank of locked window, assert(zero may be used as a default), window, error
      call MPI_WIN_LOCK(MPI_LOCK_EXCLUSIVE, 0, 0, win, ierr)

      !*** put data, # put data, data type, Process accessed by MPI_put, data address, window, error
      call MPI_PUT(local(1),1,MPI_INTEGER,0,disp_aint,1,MPI_INTEGER,win,ierr)

      !*** Completes an RMA access at the target process
      !*** rank of window, window, error
      call MPI_WIN_UNLOCK(0, win, ierr)
    endif

    !*** Finalize window
    call MPI_WIN_FREE(win,ierr)
    if(MYRANK==0) total = sum(base(1:SIZE))
    deallocate(base,local)

    !*** End testing
    call MPI_BARRIER(MPI_COMM_WORLD, ierr)
    if(MYRANK == 0) etime=MPI_Wtime()
    if(MYRANK==0)then
      write(*,"(A)") ""
      write(*,"(A)") "put"
      write(*,"(A,i4)")   "result : ", total
      write(*,"(A,E15.7)") "time   : ", etime-stime
    endif
    !***
  end subroutine OneSided_put
END PROGRAM main
```
