  updateColumns () {
    this.columnHeadsValidate()
    if (this.properties.RowSource !== '') {
      let finalWidths:Array<number> = []
      if (this.listBoxTableRef && this.listBoxTableRef.children[0] && this.listBoxTableRef.children[0].className !== 'table-body') {
        if (this.listBoxTableRef.children[0].children[0]) {
          for (let j = 0; j < this.listBoxTableRef.children[0].children[0].children.length; j++) {
            const headWidth = this.listBoxTableRef.children[0].children[0].children[j] as HTMLDivElement
            if (this.properties.ColumnCount! === -1) {
              if (j >= 0 && j < this.extraDatas.RowSourceData!.length) {
                headWidth.style.minWidth = '100px'
              }
            }
          }
        }
      }
      if (this.properties.ColumnHeads) {
        if (this.properties.ColumnWidths === '' && this.listBoxTableRef && this.listBoxTableRef.children[1]) {
          if (this.listBoxTableRef.children[1].children[0]) {
            let tempWidth
            if (this.properties.ColumnCount! <= this.extraDatas.RowSourceData![0].length) {
              if (this.properties.Width! > 100) {
                if (this.properties.ColumnCount! === -1) {
                  tempWidth = this.properties.Width! / this.extraDatas.RowSourceData![0].length
                } else {
                  tempWidth = this.properties.Width! / this.properties.ColumnCount!
                }
              } else {
                tempWidth = 100
              }
            } else {
              if (this.properties.Width! > 100) {
                tempWidth = this.properties.Width! / this.extraDatas.RowSourceData![0].length
              } else {
                tempWidth = 100
              }
            }
            for (let i = 0; i < this.listBoxTableRef.children[1].children.length; i++) {
              if (this.properties.ListStyle === 0) {
                Vue.nextTick(() => {
                  if (this.listBoxTableRef && this.listBoxTableRef.children[0] && this.listBoxTableRef.children[0].children[0]) {
                    for (let j = 0; j < this.listBoxTableRef.children[0].children[0].children.length; j++) {
                      if (this.listBoxTableRef && this.listBoxTableRef.children[0] && this.listBoxTableRef.children[0].children[0] && this.listBoxTableRef.children[0].children[0].children[j]) {
                        const headWidth = this.listBoxTableRef.children[0].children[0].children[j] as HTMLDivElement
                        if (this.properties.ColumnCount !== -1) {
                          if (j === this.listBoxTableRef.children[0].children[0].children.length - 1) {
                            if (finalWidths[j] < 10) {
                              headWidth.style.width = finalWidths[j] + 'px'
                            } else {
                              headWidth.style.width = finalWidths[j] - 4 + 'px'
                            }
                          } else {
                            headWidth.style.width = '100px'
                          }
                        }
                      }
                    }
                  }
                })
                for (let j = 0; j < this.listBoxTableRef.children[1].children[i].children.length; j++) {
                  const width = this.listBoxTableRef.children[1].children[i].children[j] as HTMLDivElement
                  if (this.properties.ColumnCount! === -1) {
                    if (j >= 0 && j < this.extraDatas.RowSourceData!.length) {
                      if (tempWidth < 10) {
                        width.style.width = tempWidth + 'px'
                      } else {
                        width.style.width = tempWidth - 4 + 'px'
                      }
                    }
                  } else if (j + 1 > this.properties.ColumnCount!) {
                    width.style.minWidth = '0px'
                    width.style.width = '0px'
                  } else {
                    if (j < this.extraDatas.RowSourceData!.length) {
                      width.style.minWidth = '100px'
                      if (tempWidth < 10) {
                        width.style.width = tempWidth + 'px'
                      } else {
                        width.style.width = tempWidth - 4 + 'px'
                      }
                    }
                  }
                }
              } else {
                Vue.nextTick(() => {
                  if (this.listBoxTableRef && this.listBoxTableRef.children[0] && this.listBoxTableRef.children[0].children[0]) {
                    for (let j = 0; j < this.listBoxTableRef.children[0].children[0].children.length; j++) {
                      if (this.listBoxTableRef && this.listBoxTableRef.children[0] && this.listBoxTableRef.children[0].children[0] && this.listBoxTableRef.children[0].children[0].children[j]) {
                        const headWidth = this.listBoxTableRef.children[0].children[0].children[j] as HTMLDivElement
                        if (this.properties.ColumnCount !== -1) {
                          if (j === this.listBoxTableRef.children[0].children[0].children.length - 1) {
                            headWidth.style.width = finalWidths[j] - 20 + 'px'
                          } else {
                            headWidth.style.width = '100px'
                          }
                        }
                      }
                    }
                  }
                })
                for (let j = 0; j < this.listBoxTableRef.children[1].children[i].children.length; j++) {
                  const width = this.listBoxTableRef.children[1].children[i].children[j] as HTMLDivElement
                  if (this.properties.ColumnCount! === -1) {
                    if (j >= 0 && j < this.extraDatas.RowSourceData!.length) {
                      width.style.width = '100px'
                    }
                  } else if (j === 1 && this.properties.ColumnCount! === 1) {
                    width.style.width = this.properties.Width! - 20 + 'px'
                  } else if (j > this.properties.ColumnCount!) {
                    width.style.minWidth = '0px'
                    width.style.width = '0px'
                  } else {
                    if (j < this.extraDatas.RowSourceData!.length) {
                      width.style.minWidth = '100px'
                      width.style.width = (this.properties.Width! / this.properties.ColumnCount!) + 'px'
                    }
                  }
                }
              }
            }
          }
        } else if (this.listBoxTableRef && this.listBoxTableRef.children[1]) {
          finalWidths = this.calculateColumnWidths()
          if (this.listBoxTableRef.children[1].children[0]) {
            for (let i = 0; i < this.listBoxTableRef.children[1].children.length; i++) {
              if (this.properties.ListStyle === 0) {
                for (let j = 0; j < this.listBoxTableRef.children[1].children[i].children.length; j++) {
                  const width = this.listBoxTableRef.children[1].children[i].children[j] as HTMLDivElement
                  if (j >= this.properties.ColumnCount! && this.properties.ColumnCount !== -1) {
                    width.style.display = 'none'
                  } else {
                    width.style.display = 'inline-block'
                    if (this.properties.ColumnCount === 1) {
                      if (finalWidths[j] === 0) {
                        width.style.display = 'none'
                      } else if (this.properties.Width! > finalWidths[0]) {
                        if (this.properties.Width! < 10) {
                          width.style.width = this.properties.Width! + 'px'
                        } else {
                          width.style.width = this.properties.Width! - 4 + 'px'
                        }
                      } else {
                        if (finalWidths[0] < 10) {
                          width.style.width = finalWidths[0] + 'px'
                        } else {
                          width.style.width = finalWidths[0] - 4 + 'px'
                        }
                      }
                    } else {
                      width.style.minWidth = '0px'
                      if (finalWidths[j] < 10) {
                        width.style.width = finalWidths[j] + 'px'
                      } else {
                        width.style.width = finalWidths[j] - 4 + 'px'
                      }
                    }
                  }
                  if (this.listBoxTableRef && this.listBoxTableRef.children[0] && this.listBoxTableRef.children[0].children[0] && this.listBoxTableRef.children[0].children[0].children[j]) {
                    const headWidth = this.listBoxTableRef.children[0].children[0].children[j] as HTMLDivElement
                    if (this.properties.ColumnCount === -1) {
                      headWidth.style.display = 'inline-block'
                      headWidth.style.minWidth = '0px'
                      if (finalWidths[j] < 10) {
                        width.style.width = finalWidths[j] + 'px'
                      } else {
                        width.style.width = finalWidths[j] - 4 + 'px'
                      }
                    } else if (j >= this.properties.ColumnCount!) {
                      headWidth.style.display = 'none'
                    } else {
                      headWidth.style.display = 'inline-block'
                      if (this.properties.ColumnCount === 1) {
                        if (finalWidths[j] === 0) {
                          headWidth.style.display = 'none'
                        } else if (this.properties.Width! > finalWidths[0]) {
                          if (this.properties.Width! < 10) {
                            headWidth.style.width = this.properties.Width! + 'px'
                          } else {
                            headWidth.style.width = this.properties.Width! - 4 + 'px'
                          }
                        } else {
                          if (finalWidths[0] < 10) {
                            headWidth.style.width = finalWidths[0] + 'px'
                          } else {
                            headWidth.style.width = finalWidths[0] - 4 + 'px'
                          }
                        }
                      } else {
                        if (finalWidths[j] < 10) {
                          headWidth.style.width = finalWidths[j] + 'px'
                        } else {
                          headWidth.style.width = finalWidths[j] - 4 + 'px'
                        }
                      }
                    }
                  }
                }
              } else {
                for (let j = 0; j < this.listBoxTableRef.children[1].children[i].children.length; j++) {
                  const width = this.listBoxTableRef.children[1].children[i].children[j] as HTMLDivElement
                  if (j > 0) {
                    if (j > this.properties.ColumnCount! && j > this.extraDatas.RowSourceData!.length - 1) {
                      width.style.display = 'none'
                    } else {
                      width.style.display = 'inline-block'
                      if (this.properties.ColumnCount === 1) {
                        if (finalWidths[j] === 0) {
                          width.style.display = 'none'
                        } else if (this.properties.Width! > finalWidths[0]) {
                          width.style.width = this.properties.Width! - 20 + 'px'
                        } else {
                          width.style.width = finalWidths[0] - 20 + 'px'
                        }
                      } else {
                        width.style.minWidth = '0px'
                        width.style.width = finalWidths[j - 1] - 20 + 'px'
                      }
                    }
                    if (this.listBoxTableRef && this.listBoxTableRef.children[0] && this.listBoxTableRef.children[0].children[0] && this.listBoxTableRef.children[0].children[0].children[j]) {
                      const headWidth = this.listBoxTableRef.children[0].children[0].children[j] as HTMLDivElement
                      if (this.properties.ColumnCount === -1) {
                        headWidth.style.display = 'inline-block'
                        headWidth.style.minWidth = '0px'
                        headWidth.style.width = finalWidths[j] - 20 + 'px'
                      } else if (j >= this.properties.ColumnCount!) {
                        headWidth.style.display = 'none'
                      } else {
                        headWidth.style.display = 'inline-block'
                        if (this.properties.ColumnCount === 1) {
                          if (finalWidths[j] === 0) {
                            headWidth.style.display = 'none'
                          } else if (this.properties.Width! > finalWidths[0]) {
                            headWidth.style.width = this.properties.Width! - 20 + 'px'
                          } else {
                            headWidth.style.width = finalWidths[0] - 20 + 'px'
                          }
                        } else {
                          headWidth.style.width = finalWidths[j] - 20 + 'px'
                        }
                      }
                    }
                  }
                }
              }
            }
          }
        }
      } else {
        if (this.properties.ColumnWidths === '' && this.listBoxTableRef && this.listBoxTableRef.children[0]) {
          if (this.listBoxTableRef.children[0].children[0]) {
            let tempWidth
            if (this.properties.ColumnCount! <= this.extraDatas.RowSourceData![0].length) {
              if (this.properties.Width! > 100) {
                if (this.properties.ColumnCount! === -1) {
                  tempWidth = this.properties.Width! / this.extraDatas.RowSourceData![0].length
                } else {
                  tempWidth = this.properties.Width! / this.properties.ColumnCount!
                }
              } else {
                tempWidth = 100
              }
            } else {
              if (this.properties.Width! > 100) {
                tempWidth = this.properties.Width! / this.extraDatas.RowSourceData![0].length
              } else {
                tempWidth = 100
              }
            }
            for (let i = 0; i < this.listBoxTableRef.children[0].children.length; i++) {
              if (this.properties.ListStyle === 0) {
                for (let j = 0; j < this.listBoxTableRef.children[0].children[i].children.length; j++) {
                  const width = this.listBoxTableRef.children[0].children[i].children[j] as HTMLDivElement
                  if (this.properties.ColumnCount! === -1) {
                    if (j >= 0 && j < this.extraDatas.RowSourceData!.length) {
                      if (tempWidth < 10) {
                        width.style.width = tempWidth + 'px'
                      } else {
                        width.style.width = tempWidth - 4 + 'px'
                      }
                    }
                  } else if (j + 1 > this.properties.ColumnCount!) {
                    width.style.minWidth = '0px'
                    width.style.width = '0px'
                  } else {
                    if (j < this.extraDatas.RowSourceData!.length) {
                      width.style.minWidth = '100px'
                      if (tempWidth < 10) {
                        width.style.width = tempWidth + 'px'
                      } else {
                        width.style.width = tempWidth - 4 + 'px'
                      }
                    }
                  }
                }
              } else {
                for (let j = 0; j < this.listBoxTableRef.children[0].children[i].children.length; j++) {
                  const width = this.listBoxTableRef.children[0].children[i].children[j] as HTMLDivElement
                  if (this.properties.ColumnCount! === -1) {
                    if (j >= 0 && j < this.extraDatas.RowSourceData!.length) {
                      width.style.width = '100px'
                    }
                  } else if (j === 1 && this.properties.ColumnCount! === 1) {
                    width.style.width = this.properties.Width! - 20 + 'px'
                  } else if (j > this.properties.ColumnCount!) {
                    width.style.minWidth = '0px'
                    width.style.width = '0px'
                  } else {
                    if (j < this.extraDatas.RowSourceData!.length) {
                      width.style.minWidth = '100px'
                      width.style.width = (this.properties.Width! / this.properties.ColumnCount!) + 'px'
                    }
                  }
                }
              }
            }
          }
        } else if (this.listBoxTableRef && this.listBoxTableRef.children[0]) {
          finalWidths = this.calculateColumnWidths()
          if (this.listBoxTableRef.children[0].children[0]) {
            for (let i = 0; i < this.listBoxTableRef.children[0].children.length; i++) {
              for (let j = 0; j < this.listBoxTableRef.children[0].children[i].children.length; j++) {
                const width = this.listBoxTableRef.children[0].children[i].children[j] as HTMLDivElement
                width.style.minWidth = '0px'
              }
              if (this.properties.ListStyle === 0) {
                for (let j = 0; j < this.listBoxTableRef.children[0].children[i].children.length; j++) {
                  const width = this.listBoxTableRef.children[0].children[i].children[j] as HTMLDivElement
                  if (j >= this.properties.ColumnCount! && this.properties.ColumnCount !== -1) {
                    width.style.display = 'none'
                  } else {
                    width.style.display = 'inline-block'
                    if (this.properties.ColumnCount === 1) {
                      if (finalWidths[j] === 0) {
                        width.style.display = 'none'
                      } else if (this.properties.Width! > finalWidths[0]) {
                        if (this.properties.Width! < 10) {
                          width.style.width = this.properties.Width! + 'px'
                        } else {
                          width.style.width = this.properties.Width! - 4 + 'px'
                        }
                      } else {
                        if (finalWidths[0] < 10) {
                          width.style.width = finalWidths[0] + 'px'
                        } else {
                          width.style.width = finalWidths[0] - 4 + 'px'
                        }
                      }
                    } else {
                      width.style.minWidth = '0px'
                      if (finalWidths[j] < 10) {
                        width.style.width = finalWidths[j] + 'px'
                      } else {
                        width.style.width = finalWidths[j] - 4 + 'px'
                      }
                    }
                  }
                }
                for (let j = 0; j < this.listBoxTableRef.children[0].children[i].children.length; j++) {
                  let colWidths = this.properties.ColumnWidths!
                  let columnWidthCount = colWidths.split(';').length
                  let totalColumnCount = this.properties.ColumnCount! < this.extraDatas.RowSourceData![0].length ? this.properties.ColumnCount! : this.extraDatas.RowSourceData![0].length
                  if (columnWidthCount > 0) {
                    if (columnWidthCount < totalColumnCount) {
                      for (let k = columnWidthCount; k < totalColumnCount!; k++) {
                        const width = this.listBoxTableRef.children[0].children[i].children[k] as HTMLDivElement
                        width.style.minWidth = '100px'
                      }
                    }
                  }
                }
              } else {
                for (let j = 0; j < this.listBoxTableRef.children[0].children[i].children.length; j++) {
                  const width = this.listBoxTableRef.children[0].children[i].children[j] as HTMLDivElement
                  if (j > 0) {
                    if (j > this.properties.ColumnCount! && j > this.extraDatas.RowSourceData!.length - 1) {
                      width.style.display = 'none'
                    } else {
                      width.style.display = 'inline-block'
                      if (this.properties.ColumnCount === 1) {
                        if (finalWidths[j] === 0) {
                          width.style.display = 'none'
                        } else if (this.properties.Width! > finalWidths[0]) {
                          width.style.width = this.properties.Width! - 20 + 'px'
                        } else {
                          width.style.width = finalWidths[0] - 20 + 'px'
                        }
                      } else {
                        width.style.minWidth = '0px'
                        width.style.width = finalWidths[j - 1] - 20 + 'px'
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
