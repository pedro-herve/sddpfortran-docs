---
layout: default
title: Home
nav_order: 1
description: "Just the Docs is a responsive Jekyll theme with built-in search that is easily customizable and hosted on GitHub Pages."
permalink: /
---

## Resume
This tutorial intends to explain how to make implement SDDP. There are some steps to follow that will be present in the next sessions.

## Introduction
SDDP is a hydrothermal dispatch model, the model calculates the least cost considering an operating policy of a hydrothermal system. The majority of SDDP code has been implemented in the FORTRAN language.
Following, are some sessions that explain important parts of SDDP code.

## Using a Module
An important struct that is frequently used is `module`. A `module`, basically can be expanded as a struct that allows importing variables, subroutines, and classes that are declared in others code place. In the next example were imported code blocks that contain some subroutines that will be used following.

```Fortran
module GerHydros
  use Factors
  use PSRClasses
  use SddpIO
end module GerHydros
```

## Creating a New Element
The first step to creating a new Element is to implement a `type` (class). Then, in the class, we must declare some attributes and methods like in the following example:

```Fortran
module GerHydros
  use <block name>
  type :: <class name>
      <attributes declaration>
      contains
          <methods declaration>
  end type <class name>
end module GerHydros
```

To optimize code we used to declare attributes using pointers. The attributes that are implemented depends on the information that will be used, following an example that make it clear:

```Fortran
module GerHydros
  use Factors
  use PSRClasses
  use SddpIO

  ! Hydro Generator data
  type :: PSRHydroGen
    integer :: size

    ! Identification
    integer         , pointer :: code(:) => NULL()
    character(len=:), pointer :: name(:) => NULL()

    ! Hydro Generator registry
    double precision, pointer :: MinTurb(:)      => NULL()     ! Minimum Turbinament
    double precision, pointer :: MaxTurb(:)      => NULL()     ! Maximum Turbinament
    double precision, pointer :: MinGene(:)      => NULL()     ! Minimum Generation
    double precision, pointer :: MaxGene(:)      => NULL()     ! Maximum Turbinament

    ! Relationships
    integer, pointer :: hid(:)     => NULL()                 ! Pointer to the hydro station
    integer, pointer :: bus(:)     => NULL()                 ! Pointer to bus

    contains
      procedure :: loadFromPSRClasses => PSRGenHydro_loadFromPSRClasses
      procedure :: validate           => PSRGenHydro_validate
  end type

end module GerHydros
```
As could be seen in the example behind, commonly  methods contains  `PSRGenHydro_loadFromPSRClasses` and `PSRGenHydro_validate` functions, both of them will be declared and explained ahead.

## Declaring a method (function)
### Methods that use functions from another block of code
Methods are functions that we declare as subroutines. Into the subroutine, it's possible to call functions that are inside others code blocks like is done in the sequent example:

```Fortran
module GerHydros
  use Factors
  use PSRClasses
  use SddpIO

  type :: <class name>
      <attributes declaration>
      contains
        procedure :: loadFromPSRClasses => PSRGenHydro_loadFromPSRClasses
        procedure :: validate           => PSRGenHydro_validate
  end type <class name>


  contains

    ! -----------------------------------------------------------------------------------------
    subroutine PSRGenHydro_loadFromPSRClasses(self)
    ! -----------------------------------------------------------------------------------------
    class(PSRHydroGen), intent(inout) :: self
    ! -----------------------------------------------------------------------------------------
    include 'PSRClasses.fi'

    ! Local
    integer*8 mapb

    ! Model
    call createMap(mapb, "PSRHydroGen")
    call totalElements("PSRHydroGen", self%size)
    call mapInt(mapb, self%code, "code")
    call mapString(mapb, self%name, "name", 12)
    
    ! Registry
    call mapReal(mapb, self%MinTurb, "MinTurb")
    call mapReal(mapb, self%MaxTurb, "MaxTurb")
    call mapReal(mapb, self%MinGene, "MinGene")
    call mapReal(mapb, self%MaxGene, "MaxGene")
    call finishMap(mapb)

    ! Relationships
    call mapRelationShip("PSRHydroGen","PSRHydro",self%hid, &
                          PSR_RELATIONSHIP_1TO1)
    call mapIndirectRelationShip("PSRHydroGen", &
                                 "PSRSystem", &
                                 "PSRBus", &
                                 self%bus,PSR_RELATIONSHIP_1TO1)

    ! Initial data validation
    call self%validate

    return
    end subroutine

end module GerHydros
```

Should be noted that the subroutine is started with `contains` to indicate that the next code lines are related to methods that were declared in a previous class. It's important to be secure that the functions called are inside code blocks declared in the module.

### Methods that use routines to print messages on SDDP prompt
Some methods ( principally evaluate ones) made some prints on the program prompt. The messages that should be printed, needs to be written in consecutive files `sddppor.fmt`, `sddpesp.fmt` and `sddpeng.fmt`, the messages should be written in Portuguese, Spain, and English respectively. Following the example of routine messages in English.

```Fortran
***ROTINA: gerhid
01 (' Error: Hydro Generation Unit {}: duplicated code')
02 (' Error: Hydro Generation Unit {}: duplicated name')
03 (' Error: Hydro Generation Unit {}: MinTurb is not avaleble')
04 (' Error: Hydro Generation Unit {}: MaxTurb is not avaleble')
05 (' Error: Hydro Generation Unit {}: MinGene is not avaleble')
06 (' Error: Hydro Generation Unit {}: MaxGene is not avaleble')
07 (' Error: Hydro Generation Unit {}: MaxGene < MinGene')
08 (' Error: Hydro Generation Unit {}: MaxTurb < MinTurb')
09 (/,1x,'Please, check confguration data.')
```
Note that in the behind example 9 messages could be printed on the SDDP prompt, these messages are numbered from 01 to 09. The numbering is important because that is the way to correlate some occurrence with his respective message. The next example shows a method that prints messages on the prompt. Something important to noted is that the routine name is declared in the first line of the example code (`***ROTINA: gerhid`).

```Fortran
    !------------------------------------------------------------------------------------------
    subroutine PSRGenHydro_validate(self,iper)
    !------------------------------------------------------------------------------------------
    ! 
    ! -----------------------------------------------------------------------------------------
    class(PSRHydroGen), intent(in)           :: self
    integer           , intent(in), optional :: iper
    !------------------------------------------------------------------------------------------
    ! Local
    integer :: i, k, jpxa, jyra
    logical :: is_mod
    character(:), allocatable :: routine

    routine = 'gerhid'

    is_mod = present(iper)
    do i = 1, self%size
      call search(self%code,self%size,self%code(i),k)
      if( k /= i ) then
        call sddp_io%error(routine,1, [ arg(self%name(i)) ] )
      end if
      call searca(self%name,self%size,self%name(i),k)
      if( k /= i ) then
        call sddp_io%error(routine,2, [ arg(self%code(i)) ] )
      end if
      if( self%MinTurb(i) < rzero ) then
        call sddp_io%error(routine,3, [ arg(self%name(i)) ] )
      end if
      if( self%MaxTurb(i) < rzero ) then
        call sddp_io%error(routine,4, [ arg(self%name(i)) ] )
      end if
      if( self%MinGene(i) < rzero ) then
        call sddp_io%error(routine,5, [ arg(self%name(i)) ] )
      end if
      if( self%MaxGene(i) < rzero ) then
        call sddp_io%error(routine,6, [ arg(self%name(i)) ] )
      end if
      if( self%MaxGene(i) < self%MinGene(i) ) then
        call sddp_io%error(routine,7, [ arg(self%name(i)) ] )
      end if
      if( self%MaxTurb(i) < self%MinTurb(i) ) then
        call sddp_io%error(routine,8, [ arg(self%name(i)) ] )
      end if
    end do

    call sddp_io%error_stop(routine,9)

    return
    end subroutine
```
Observe that the routine name has been declared like it was named in the  message files that were presented previously. The routine `sddp_io` is used to print the message and one of its parameters is the number of message name that needs to be printed.

Next, one complete example code.
```Fortran
!==============================================================================================
! PSR, Stochastic Dual Dynamic Programming (SDDP)
!==============================================================================================
!
! MODULE: 
!
! DESCRIPTION:
!>   
!
!==============================================================================================
module CspPlant
  use Factors
  use PSRClasses
  use SddpIO

  implicit none

  ! Hydro Generator data
  type :: PSRHydroGen
    integer :: size

    ! Identification
    integer         , pointer :: code(:) => NULL()
    character(len=:), pointer :: name(:) => NULL()

    ! Hydro Generator registry
    double precision, pointer :: minturb(:)      => NULL()     ! Minimum Turbinament
    double precision, pointer :: maxturb(:)      => NULL()     ! Maximum Turbinament
    double precision, pointer :: mingene(:)      => NULL()     ! Minimum Generation
    double precision, pointer :: maxgene(:)      => NULL()     ! Maximum Turbinament

    ! Relationships
    integer, pointer :: hid(:)     => NULL()                 ! Pointer to the hydro station
    integer, pointer :: bus(:)     => NULL()                 ! Pointer to bus

    contains
      procedure :: loadFromPSRClasses => PSRGenHydro_loadFromPSRClasses
      procedure :: validate           => PSRGenHydro_validate
  end type

  type(PSRHydroGen) :: unit

  contains

    ! -----------------------------------------------------------------------------------------
    subroutine PSRGenHydro_loadFromPSRClasses(self)
    ! -----------------------------------------------------------------------------------------
    class(PSRHydroGen), intent(inout) :: self
    ! -----------------------------------------------------------------------------------------
    include 'PSRClasses.fi'

    ! Local
    integer*8 mapb

    ! Model
    call createMap(mapb, "PSRHydroGen")
    call totalElements("PSRHydroGen", self%size)
    call mapInt(mapb, self%code, "code")
    call mapString(mapb, self%name, "name", 12)
    
    ! Registry
    call mapReal(mapb, self%minturb, "minturb")
    call mapReal(mapb, self%maxturb, "maxturb")
    call mapReal(mapb, self%mingene, "mingene")
    call mapReal(mapb, self%maxgene, "maxgene")
    call finishMap(mapb)

    ! Relationships
    call mapRelationShip("PSRHydroGen","PSRHydro",self%hid, &
                          PSR_RELATIONSHIP_1TO1)
    call mapIndirectRelationShip("PSRHydroGen", &
                                 "PSRSystem", &
                                 "PSRBus", &
                                 self%bus,PSR_RELATIONSHIP_1TO1)

    ! Initial data validation
    call self%validate

    return
    end subroutine

    !------------------------------------------------------------------------------------------
    subroutine PSRGenHydro_validate(self,iper)
    !------------------------------------------------------------------------------------------
    ! 
    ! -----------------------------------------------------------------------------------------
    class(PSRHydroGen), intent(in)           :: self
    integer           , intent(in), optional :: iper
    !------------------------------------------------------------------------------------------
    ! Local
    integer :: i, k, jpxa, jyra
    logical :: is_mod
    character(:), allocatable :: routine

    routine = 'gerhid'

    is_mod = present(iper)
    do i = 1, self%size
      call search(self%code,self%size,self%code(i),k)
      if( k /= i ) then
        call sddp_io%error(routine,1, [ arg(self%name(i)) ] )
      end if
      call searca(self%name,self%size,self%name(i),k)
      if( k /= i ) then
        call sddp_io%error(routine,2, [ arg(self%code(i)) ] )
      end if
      if( self%minturb(i) < rzero ) then
        call sddp_io%error(routine,3, [ arg(self%name(i)) ] )
      end if
      if( self%maxturb(i) < rzero ) then
        call sddp_io%error(routine,4, [ arg(self%name(i)) ] )
      end if
      if( self%mingene(i) < rzero ) then
        call sddp_io%error(routine,5, [ arg(self%name(i)) ] )
      end if
      if( self%maxgene(i) < rzero ) then
        call sddp_io%error(routine,6, [ arg(self%name(i)) ] )
      end if
      if( self%maxgene(i) < self%mingene(i) ) then
        call sddp_io%error(routine,7, [ arg(self%name(i)) ] )
      end if
      if( self%maxturb(i) < self%minturb(i) ) then
        call sddp_io%error(routine,8, [ arg(self%name(i)) ] )
      end if
    end do

    call sddp_io%error_stop(routine,9)

    return
    end subroutine
end module GerHydros
!==============================================================================================

```


