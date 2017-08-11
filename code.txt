package main

import (
	"bufio"
	"encoding/csv"
	"flag"
	"fmt"
	"math"
	"os"
	"sort"
	"strconv"
)

//Dataset .
type Dataset struct {
	FileName string
	FileType int
}

//Dino .
type Dino struct {
	Name         string
	StrideLength float64
	LegLength    float64
	Stance       string
	Diet         string
	Speed        float64
}

//Dinos .
type Dinos []Dino

var (
	dataset1 = flag.String("dataset1", os.Getenv("DINOS_DS1"), "Dataset 1")
	dataset2 = flag.String("dataset2", os.Getenv("DINOS_DS2"), "Dataset 2")
	datasets []Dataset
)

func parseFlags() {
	//args parser and validate
	flag.Parse()
	if *dataset1 == "" || *dataset2 == "" {
		flag.PrintDefaults()
		os.Exit(1)
	} else {
		//fileType
		// 1 - dataset1
		// 2 - dataset2
		datasets = []Dataset{{FileName: *dataset1, FileType: 1}, {FileName: *dataset2, FileType: 2}}
	}
}

func init() {
	parseFlags()
}

func main() {
	dinos := Dinos{}
	dinos.loadCSV(datasets)
	dinos = dinos.selectByStance("bipedal")
	dinos.speedCalculate()
	dinos.orderByDesc()

	for _, dino := range dinos {
		if dino.Speed > 0 {
			fmt.Printf("%v\n", dino.Name)
		}
	}
}

//Load datasets from csv.
func (d *Dinos) loadCSV(ds []Dataset) {
	//Tmporary map of Dino
	tmpDinos := make(map[string]*Dino)
	for _, dataset := range ds {
		file := dataset.FileName
		fileType := dataset.FileType

		// Open and read the dataset from CSV to slace of lines
		csvFile, _ := os.Open(file)
		csv := csv.NewReader(bufio.NewReader(csvFile))
		lines, _ := csv.ReadAll()
		csvFile.Close()

		// Reading dataset lines (slace lines)
		for i, line := range lines {
			if i == 0 {
				// skip header line
				continue
			}
			if tmpDinos[line[0]] == nil {
				tmpDinos[line[0]] = &Dino{}
			}
			tmpDinos[line[0]].Name = line[0]
			//fileType
			// 1 - dataset1
			// 2 - dataset2
			switch fileType {
			case 1:
				//[0]NAME,[1]LEG_LENGTH,[2]DIET	     	===> File1
				tmpDinos[line[0]].LegLength, _ = strconv.ParseFloat(line[1], 64)
				tmpDinos[line[0]].Diet = line[2]
			case 2:
				//[0]NAME,[1]STRIDE_LENGTH,[2]STANCE  ===> File2
				tmpDinos[line[0]].StrideLength, _ = strconv.ParseFloat(line[1], 64)
				tmpDinos[line[0]].Stance = line[2]
			}
		}
	}
	for key := range tmpDinos {
		*d = append(*d, *tmpDinos[key])
	}
}

//Calculate speed of the Dinos
func (d *Dinos) speedCalculate() {
	var speed, g float64
	g = 9.8
	//Calculate speed
	for i, obj := range *d {
		//Only calculate if Stride Length and Leg Length are greater than zero
		if obj.StrideLength > 0 && obj.LegLength > 0 {
			speed = ((obj.StrideLength / obj.LegLength) - 1) * math.Sqrt(obj.LegLength*g)
			(*d)[i].Speed = speed
		}
	}
}

//Filter Dinos by Stance type
func (d *Dinos) selectByStance(stance string) (outD Dinos) {
	outD = make(Dinos, 0)
	for _, dino := range *d {
		if dino.Stance == stance {
			outD = append(outD, dino)
		}
	}
	return outD
}

//Sorting slace of Dino or type Dinos
func (d *Dinos) orderByDesc() {
	sort.Sort(sort.Reverse(*d))
}

//Sorting Helpers
func (d Dinos) Len() int           { return len(d) }
func (d Dinos) Less(i, j int) bool { return d[i].Speed < d[j].Speed }
func (d Dinos) Swap(i, j int)      { d[i], d[j] = d[j], d[i] }
