# topcoder_databricks_competition
guruInMilkyway's code for databricks competition by topcoder

Please download the zip and open each html in your brower (I have Tested it with Chrome and Edge

/
KDB_solutions_Q1
*********Question 1: Getting Non-zero Rows ****************
Han, Yang 2:22 PM: 

quick q question - do you know how to select columns with at least one row has non 0 value?

Han, Yang 2:29 PM: 

t:([]a:100?til 10;b:100?til 10;c:0),([]a:enlist 0;b:enlist 0;c:enlist 0) 
\

/t-table definition
t:([]a:100?til 10;b:100?til 10;c:0),([]a:enlist 0;b:enlist 0;c:enlist 0) 
t
/vanilla filter
select from (select from (select from t where c=0) where b=0) where a=0

/ Approach-1: Function for 3 rows - hardcoded ******:
remove_zero_rows_for3: {[t] t except select from (select from (select from t where c=0) where b=0) where a=0}
t_with_noallzero_rows: remove_zero_rows_for3[t]
t_with_noallzero_rows

/ Approach-2: False Logic :-) ******:
remove_zero_rows_for3_using_sum:{[t]select a,b,c from (update d:a+b+c from `t) where d<>0}
t_with_noallzero_rows:remove_zero_rows_for3_using_sum[t]
t_with_noallzero_rows

/ Approach-3 (Dynamic approach): Generic case for varying number of cols (with functional form) ******:
remove_zero_rows: {[t] t except ?[t; {(not;x)} each cols t; 0b; ()]}

/Testing with t
t:([]a:100?til 10;b:100?til 10;c:0),([]a:enlist 0;b:enlist 0;c:enlist 0) 
t_with_noallzero_rows: remove_zero_rows[t]
t_with_noallzero_rows

t:([]a:100?til 10;b:100?til 10;c:0; d:0; e:0),([]a:enlist 0;b:enlist 0;c:enlist 0; d:0; e:0) 
t_with_noallzero_rows: remove_zero_rows[t]
t_with_noallzero_rows

/Testing with p
p:([]a:10?10;b:10?100;c:0; d:0; e:0),([] a:enlist 0;b:enlist 0;c:enlist 0; d:enlist 0; e:enlist 0) 
p
t_with_noallzero_rows: remove_zero_rows[p]
t_with_noallzero_rows

/corner_case
p_neg:([]a:-1 1 0;b:-5 5 0;c:0; d:0; e:0),([] a:enlist 0;b:enlist 0;c:enlist 0; d:enlist 0; e:enlist 0) 
p_neg
t_with_noallzero_rows: remove_zero_rows[p_neg]
t_with_noallzero_rows

/
*********Question 2: Getting Non-zero Columns ****************
\

/Testing with t
t:([]a:100?til 10;b:100?til 10;c:0),([]a:enlist 0;b:enlist 0;c:enlist 0) 
m:([]a:100?til 10;d:0;e:0; b:100?til 10;c:0),([]a:enlist 0;d:0; e:0; b:enlist 0;c:enlist 0) 

/ Approach-1: With Dict **********: 
get_non_zero_cols: { [p]                    
                    d_p: flip p;
                    f_p: (and/) p=0;
                    col: (f_p?1b);                    
                    while[((col in key f_p) = 1b);                        
                        f_p: col _ f_p;
                        d_p: col _ d_p;
                        col: (f_p?1b)                        
                    ];                
                    flip d_p
                    }

table_with_non_zero_columns_t: get_non_zero_cols[t]
table_with_non_zero_columns_m: get_non_zero_cols[m]
table_with_non_zero_columns_t
table_with_non_zero_columns_m

/ Approach-2 (Sub optimal) with tranpose matrix - faced problems with naming of columns *******:
get_table_transpose:{[t] (flip ((count t)?(cols t))! flip value flip t)};

p:m
p
trans_p:get_table_transpose[p];
trans_p
result_trans: remove_zero_rows[trans_p];
result_trans
table_with_nonzero_columns: get_table_transpose[result_trans];
table_with_nonzero_columns

/////////////////////////////////////
kdb_solutions_Q2_v1
/Q-2:
 Input:11100110001001 -> Function ??? -> Output:11100220003004
\

/This function updates the list by tracking 0 to 1 changes in thelist 
f_in:{[L]     
    
    /catch 0->1 changes in the list
    index: where (deltas L)=1; index: index, count L; $[first L = 0; [;]; [index: 1_ index;]];
    
    i:0;n:1;
    do[(count index) - 1;          
        /Filter elements = 1 that need to be updated. 
        L[index[i] + where L[index[i] + til (index[i+1] - index[i])]=1]+:(n+i);                
        i+:1;       
    ];   
    L
}

/considering the corner cases with 1 elements in the list where delta fails. Hence handled it through another function
update_list:{[L]
    $[ (count L) > 2;[L: f_in[L];];];L
}

L0:0
uL0: update_list[L0]
uL0

L1:1
uL1: update_list[L1]
uL1

L2: 1 1 1 0 0 1 1 0 0 0 1 0 0 1
uL2: update_list[L2]
uL2

L3: 1 1 1 0 0 1 1 0 0 0 1 0 0 1 1 0 1 0 0 1 1 1 1
uL3: update_list[L3]
uL3

L4: 0 0 1 1 1 1 0 1 0 0 0 1 0 0 
uL4: update_list[L4]   
uL4

//////////////////////////////////////////
KDB_solutions_Q2_v2

/Q-2:
 Input:11100110001001 -> Function ??? -> Output:11100220003004
\

l:1 1 1 0 0 1 1 0 0 0 1 0 0 1;

split_lists: (where (deltas l)=1) cut l; 
split_lists;

update_ones_in_list: {[p;l] l[where l=1]:p;l};

p:0;
update_ones:{`p set p+:1; split_lists[p-1]: update_ones_in_list[p; split_lists[p-1]]; l_out[p-1]};

update_ones each split_lists;
outList: raze split_lists;
outList;


