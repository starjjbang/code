### R Packages

To take advantage of the optimized configuration settings included with this template, you must have the [`renv` package](https://rstudio.github.io/renv/articles/renv.html) installed on your local computer. While in an active RStudio session, simply run the following line of code in the Console.

```r
rm(list=ls())
  #https://ycl6.github.io/GO-Enrichment-Analysis-Demo/4_enrichR.html
  p_dat = read.csv('/home/starjjbang/Deep_drug/5_GSEA/protein_data.txt',sep='\t')
  p_dat1 = p_dat[which(p_dat$pfdr < 0.05),]
```

#### Clone this template

* Clone this repository:

  ```bash
  rm(list=ls())
  #https://ycl6.github.io/GO-Enrichment-Analysis-Demo/4_enrichR.html
  p_dat = read.csv('/home/starjjbang/Deep_drug/5_GSEA/protein_data.txt',sep='\t')
  p_dat1 = p_dat[which(p_dat$pfdr < 0.05),]
  ```

* Run the initializer script:

  ```bash
  ./init.sh
  ```




g_dat = read.csv('/home/starjjbang/Deep_drug/1_data/10_drug_score/gene_score_t_test.txt',sep='\t')
g_dat1 = g_dat[which(g_dat$p.value< 0.05),]

r_dat = read.csv('/home/starjjbang/Deep_drug/5_GSEA/TCGA_BRCA_RNA_data.txt',sep='\t')
r_dat1 = r_dat[which(r_dat$PValue < 0.05),]

dim(p_dat1);dim(g_dat1);dim(r_dat1)

library("enrichR")
dbs <- listEnrichrDbs()
dbs <- dbs[order(dbs$libraryName),]
dbs$libraryName[grep("KEGG",dbs$libraryName)]

term_ext <-function(db_type, pos){
  d_Enriched <- enrichr(genes = g_lst1$SYMBOL, databases = db_type)
  in_dat = d_Enriched[[pos]]
  tn = in_dat[which(in_dat$P.value < 0.05),] %>% select(Term)
  return(tn)
}

enrich_fun <-function(id_lst,id_type,db_type){
  p_g_lst = clusterProfiler::bitr(id_lst, fromType = id_type, toType = "SYMBOL", OrgDb = "org.Hs.eg.db")
  t_Enriched_go <- enrichr(genes = p_g_lst$SYMBOL, databases = db_type)
  in_dat = t_Enriched_go[[1]]
  tn = in_dat[which(in_dat$P.value < 0.05),] %>% select(Term)
  return(tn)
  
}

enrich_fun1 <-function(id_lst,id_type,db_type){
  p_g_lst = clusterProfiler::bitr(id_lst, fromType = id_type, toType = "SYMBOL", OrgDb = "org.Hs.eg.db")
  t_Enriched_go <- enrichr(genes = p_g_lst$SYMBOL, databases = db_type)
  in_dat = t_Enriched_go[[1]]
  tn = in_dat[which(in_dat$P.value < 0.05),]
  return(tn)
  
}

path_db = "KEGG_2021_Human" #"MSigDB_Hallmark_2020"

p_mf = enrich_fun(rownames(p_dat1),"ENSEMBL","GO_Molecular_Function_2023")
p_cc = enrich_fun(rownames(p_dat1),"ENSEMBL","GO_Cellular_Component_2023")
p_bp = enrich_fun(rownames(p_dat1),"ENSEMBL","GO_Biological_Process_2023")
p_path = enrich_fun(rownames(p_dat1),"ENSEMBL",path_db)

r_mf = enrich_fun(rownames(r_dat1),"ENSEMBL","GO_Molecular_Function_2023")
r_cc = enrich_fun(rownames(r_dat1),"ENSEMBL","GO_Cellular_Component_2023")
r_bp = enrich_fun(rownames(r_dat1),"ENSEMBL","GO_Biological_Process_2023")
r_path = enrich_fun(rownames(r_dat1),"ENSEMBL",path_db)

g_mf = enrich_fun(g_dat1$id,"ENSEMBL","GO_Molecular_Function_2023")
g_cc = enrich_fun(g_dat1$id,"ENSEMBL","GO_Cellular_Component_2023")
g_bp = enrich_fun(g_dat1$id,"ENSEMBL","GO_Biological_Process_2023")
g_path = enrich_fun1(g_dat1$id,"ENSEMBL",path_db)


d_list = list.files('/home/starjjbang/Deep_drug/5_GSEA/bc target/')

gene_lst = c()
for(i in 1:length(d_list)){
  d_dat = read.csv(paste0('/home/starjjbang/Deep_drug/5_GSEA/bc target/',d_list[i]),sep=',') %>% 
    select(Target.GeneID, Activity.Value..uM.)
  d_dat1 = d_dat[!is.na(d_dat$Target.GeneID),] %>% unique()
  d_dat1$Target.GeneID = as.character(d_dat1$Target.GeneID)
  gene_lst = c(gene_lst, d_dat1$Target.GeneID)
}
length(gene_lst); length(unique(gene_lst))

dbs_go = c("GO_Molecular_Function_2023","GO_Cellular_Component_2023","GO_Biological_Process_2023")
length(d_list)
go_bp = c()
go_mf = c()
go_cc = c()
kegg = c()
for(i in 1:length(d_list)){
  d_dat = read.csv(paste0('/home/starjjbang/Deep_drug/5_GSEA/bc target/',d_list[i]),sep=',') %>% 
    select(Target.GeneID, Activity.Value..uM.)
  d_dat1 = d_dat[!is.na(d_dat$Target.GeneID),] %>% unique()
  d_dat1$Target.GeneID = as.character(d_dat1$Target.GeneID)
  #tmp_sc = d_dat1$Activity.Value..uM.[!is.na(d_dat1$Activity.Value..uM.)]
  #rs= max(tmp_sc) + 1 - tmp_sc
  #names(rs) = tmp_sc
  #rs_score = sapply(d_dat1$Activity.Value..uM.,function(x) 
  #{ifelse(any(!is.na(x)), unique(rs[which(names(rs)== x)]), 1 )})
  #d_dat1$Activity.Value..uM. = rs_score
  colnames(d_dat1) = c("ID")
  if(nrow(d_dat1)> 1){
    g_lst1 = clusterProfiler::bitr(d_dat1$ID, fromType = "ENTREZID", toType = "SYMBOL", OrgDb = "org.Hs.eg.db")
    go_mf =c(go_mf, term_ext(dbs_go,1))
    go_cc = c(go_cc, term_ext(dbs_go,2))
    go_bp = c(go_bp, term_ext(dbs_go,3))
    kegg = c(kegg, term_ext(path_db,1))    
  }

}

cal_rate <-function(in_dat, ref_dat,num_gene){
  inter = length(intersect(unlist(in_dat),unique(unlist(ref_dat))))
  rt = inter/num_gene
  return(rt)
}


g_dat = data.frame(GS = c("BP(GO)","MF(GO)","CC(GO)","KEGG"),
                   
                   CGF = c(cal_rate(g_bp, go_bp, nrow(g_dat1)),
                           cal_rate(g_mf, go_mf, nrow(g_dat1)),
                           cal_rate(g_cc, go_cc, nrow(g_dat1)),
                           cal_rate(g_path, kegg, nrow(g_dat1))),
                   
                   RNA = c(cal_rate(r_bp, go_bp, nrow(r_dat1)),
                           cal_rate(r_mf, go_mf, nrow(r_dat1)),
                           cal_rate(r_cc, go_cc, nrow(r_dat1)),
                           cal_rate(r_path, kegg, nrow(r_dat1))),
                   
                   Protein = c(cal_rate(p_bp, go_bp, nrow(p_dat1)),
                               cal_rate(p_mf, go_mf, nrow(p_dat1)),
                               cal_rate(p_cc, go_cc, nrow(p_dat1)),
                               cal_rate(p_path, kegg, nrow(p_dat1)))
)
g_dat


g_dat$GS = c("BP(C5)","MF(C5)","CC(C5)","KEGG(C2)")
colnames(g_dat) = c("lab","CGF","RNA","Protein")
g_dat = melt(g_dat, id = c("lab"))
g_dat$value =round(as.numeric(g_dat$value),3)

p = ggplot(g_dat, aes(fill=variable, y=value, x=variable)) + 
  geom_bar(position="dodge", stat="identity") + 
  facet_wrap(~ lab,ncol=4, scales = "free") + 
  theme(panel.background = element_blank(),
        panel.border = element_rect(color = "black", fill = NA, size = 0.4)) + 
  xlab("") + ylab("Normalized rate of intersection terms") + 
  scale_fill_manual(values=c("#9EA3AF","#E6D6BE","#D9ABA3")) + 
  NoLegend()
p

fig_dir = '/home/starjjbang/Deep_drug/6_FIg/1_Fig1/'
png(filename = paste0(fig_dir,"8_GSEA.png"), width = 3500, height = 1000,
    units = "px", bg = "white", res = 300)
print(p)
dev.off()
#################################################################################################
term_check<-function(in_dat,i){
  if(length(which(in_dat$Term == sel_path[i])) > 0){
    tn = in_dat[which(in_dat$Term == sel_path[i]),] %>% select(Combined.Score)  
  }else{
    tn = 0
  }
  return(tn)
}

sel_path = c("Ras signaling pathway" , "Estrogen signaling pathway","cAMP signaling pathway","T cell receptor signaling pathway"  ,
  "B cell receptor signaling pathway" ,"Chemokine signaling pathway" )
g_path = enrich_fun1(g_dat1$id,"ENSEMBL",path_db)
p_path = enrich_fun1(rownames(p_dat1),"ENSEMBL",path_db)
r_path = enrich_fun1(rownames(r_dat1),"ENSEMBL",path_db)
for_tmp = c()
for(i in 1:length(sel_path)){
  for_tmp = rbind(for_tmp,unlist(c(term_check(g_path),term_check(r_path),term_check(p_path))))
}
rownames(for_tmp) = sel_path
colnames(for_tmp) = c("CGF","RNA","Protein")




enrich_fun <-function(id_lst,id_type,db_type,dt){
  re_dir = '/home/starjjbang/Deep_drug/5_GSEA/GSEA_result/'
  p_g_lst = clusterProfiler::bitr(id_lst, fromType = id_type, toType = "SYMBOL", OrgDb = "org.Hs.eg.db")
  t_Enriched_go <- enrichr(genes = p_g_lst$SYMBOL, databases = db_type)
  g_tmp = data.frame(t_Enriched_go)
  write.table(g_tmp, paste0(re_dir,db_type,'_',dt,'.txt'),sep='\t',quote = FALSE,row.names = FALSE)
  return(g_tmp)
}

enrich_fun1 <-function(id_lst,id_type,db_type,dt){
  p_g_lst = clusterProfiler::bitr(id_lst, fromType = id_type, toType = "SYMBOL", OrgDb = "org.Hs.eg.db")
  t_Enriched_go <- enrichr(genes = p_g_lst$SYMBOL, databases = db_type)
  g_tmp = data.frame(t_Enriched_go)
  write.table(g_tmp, paste0(re_dir,db_type,'_',dt,'.txt'),sep='\t',quote = FALSE,row.names = FALSE)
}

path_db = "KEGG_2021_Human" #"MSigDB_Hallmark_2020"

p_mf = enrich_fun(rownames(p_dat1),"ENSEMBL","GO_Molecular_Function_2023","P")
p_cc = enrich_fun(rownames(p_dat1),"ENSEMBL","GO_Cellular_Component_2023","P")
p_bp = enrich_fun(rownames(p_dat1),"ENSEMBL","GO_Biological_Process_2023","P")
p_path = enrich_fun(rownames(p_dat1),"ENSEMBL",path_db,"P")

r_mf = enrich_fun(rownames(r_dat1),"ENSEMBL","GO_Molecular_Function_2023","R")
r_cc = enrich_fun(rownames(r_dat1),"ENSEMBL","GO_Cellular_Component_2023","R")
r_bp = enrich_fun(rownames(r_dat1),"ENSEMBL","GO_Biological_Process_2023","R")
r_path = enrich_fun(rownames(r_dat1),"ENSEMBL",path_db,"R")

g_mf = enrich_fun(g_dat1$id,"ENSEMBL","GO_Molecular_Function_2023","G")
g_cc = enrich_fun(g_dat1$id,"ENSEMBL","GO_Cellular_Component_2023","G")
g_bp = enrich_fun(g_dat1$id,"ENSEMBL","GO_Biological_Process_2023","G")
g_path = enrich_fun1(g_dat1$id,"ENSEMBL",path_db,"G")
